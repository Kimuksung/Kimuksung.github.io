---
layout: post
title:  "Kafka 데이터 실시간 스트리밍하기 with HDFS"
author: Kimuksung
categories: [ Spark, Hadoop, Kafka, Cluster ]
tags: [Spark, Hadoop, Kafka, Cluster]
image: assets/images/emr_spark.png
comments: False
featured: true
---

지금까지, 로그 데이터 구성하기를 진행하는 과정에 있어 Kafka, HDFS, Spark 클러스터를 구성하였습니다.

Kafka의 test Topic을 Spark에서 Consume하여 HDFS에 Parquet로 실시간 업로드하는 과정을 구성해보겠습니다.

<br>

##### 1. Spark에서 Hadoop Cluster와 Kafka Cluster를 연결하기에 SparkSession이 다른 config 값을 구성하는데 어떻게 만들어야할까?
---
- 처음 접근은 kafka 연결 세션과 Hadoop 연결 세션을 만들어 구성한다고 생각하였다.
- Kafka로부터는 Consume / HDFS에는 업로드
- 코드 구성 과정에서 알게 된점
    - HDFS에 접근하기 위해서는 별도의 config 값이 없어도 된다.
    - Kafka 연결 세션만 만들어주면 된다.

```python
from pyspark.sql import SparkSession

scala_version = '2.12'
spark_version = '3.1.1'
packages = [
    f'org.apache.spark:spark-sql-kafka-0-10_{scala_version}:{spark_version}',
    'org.apache.kafka:kafka-clients:3.2.0'
]

spark = SparkSession.builder\
    .master("local")\
    .appName("kafka-example")\
    .config("spark.jars.packages", ",".join(packages))\
    .getOrCreate()
```


##### 2. 실시간으로 처리하기 위해서는 어떤 코드를 구성해서 동작시켜야 할까?
---
- Spark에서는 `readStream`과 `writeStream`을 지원
- Stream 함수를 쓴 경우에는 이와 관련된 변수는 stream만 동작한다. ( 에러 발생 )
- readStream은 Kafka Consume 시 사용
- writeStream은 HDFS에 값을 저장 할 때 사용
- format에 따라 출력값을 변경 할 수 있다. ( console, parquet .. )
- outputMode에는 append, update 등이 있으니 필요한 경우에 맞춰 사용하면 된다. 저의 경우에는 로그 데이터를 계속하여 추가할 생각이었기에 append로 구성
- path는 `file`, `hdfs`, `s3` 등으로 표현 가능
- stream 사용 시 별도의 종료 커맨드가 없다면 멈출 수 없기에 `awaitTermination`를 사용하여 멈추어 준다.

```python
# Kafka Consume Stream
kafkaDf = spark.readStream.format("kafka")\
  .option("kafka.bootstrap.servers", "KafkaBroker:9092")\
  .option("subscribe", 'test')\
  .option("startingOffsets", "earliest")\
  .load()

query = kafkaDf.selectExpr("CAST(value AS STRING)") \
.writeStream \
.format("console") \
.option("truncate", "false") \
.start()

# Write stream - HDFS
query2 = kafkaDf.selectExpr("CAST(value AS STRING)") \
.writeStream \
.format("parquet") \
.outputMode("append") \
.option("checkpointLocation", "/check") \
.option("path", f"hdfs://hadoop_ip:port/{filedirectory}") \
.start()

query.awaitTermination()
query2.awaitTermination()
```


##### 3. 실시간 처리 중인데 데이터가 중간 값부터만 저장이 될까?
---
- 위의 코드로 로그 데이터를 Produce 후 다시 업로드하는데, `startingOffsets` = `earliest` 임에도 불구하고 처음부터 가져오지 못함
- `checkpointLocation` 값은 현재 저장 할 값에 대해 연결 끊김이나 여러 이슈 사항에 대비하여 offset을 저장
- 아래와 같이 check 파일을 확인하여 보면 offsets 값이 표현되어 해당 값 이후 부터 가져온다.

```python
$ docker exec -it jupyter /bin/bash
$ cd /check
$ ls
```

<a href="https://imgbb.com/"><img src="https://i.ibb.co/PDpdYKS/2023-10-13-12-05-00.png" alt="2023-10-13-12-05-00" border="0"></a>

<br>

##### 4. Parquet가 0부터 저장되지 않는다면 어떤 일이 발생할까?
---
- HDFS로 부터 데이터 정보를 읽으려고 하는데 읽지 못하는 에러가 발생한다.
- `checkpointLocation` 의 값을 삭제 후, 처음 부터 다시 적재해준다.

<a href="https://ibb.co/GJw8w8P"><img src="https://i.ibb.co/X4BKBKJ/2023-10-13-12-09-34.png" alt="2023-10-13-12-09-34" border="0"></a>

```python
class ConnectHadoop:
    def __init__(self, ip="", port="", file_path="", file_name="", user_name="kim"):
        self.hdfs_ip = ip
        self.hdfs_port = port
        self.hdfs_server = f"hdfs://{self.hdfs_ip}:{self.hdfs_port}"
        self.hdfs_file_path = file_path
        self.hdfs_file_name = file_name
        self.hdfs_user_name = user_name
        self.spark = SparkSession.builder \
            .appName("WriteHDFS") \
            .config("spark.hadoop.fs.defaultFS", self.hdfs_server) \
            .config("spark.hadoop.yarn.resourcemanager.hostname", self.hdfs_ip) \
            .config("spark.hadoop.user.name", self.hdfs_user_name) \
            .getOrCreate()

    def __del__(self):
        print('end spark')

    def read_file(self, read_type):
        if read_type == "txt":
            return self.spark.read.text(self.hdfs_file_path + self.hdfs_file_name)
        elif read_type == "parquet":
            return self.spark.read.parquet(self.hdfs_file_path)
        elif read_type == "csv":
            return self.spark.read.csv(self.hdfs_file_path)

if __name__ == "__main__":
    hadoop = ConnectHadoop(file_path=f"/test/log_dir/2023-10-12")
    data = hadoop.read_file(read_type="parquet")
    data.show(data.count(), False)
```
