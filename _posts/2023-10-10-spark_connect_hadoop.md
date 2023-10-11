---
layout: post
title:  "Spark - Hadoop 연결하기"
author: Kimuksung
categories: [ Spark, Hadoop, Ec2, Cluster ]
tags: [Spark, Hadoop, Ec2, Cluster]
image: assets/images/emr_spark.png
comments: False
featured: true
---

Pyspark로 연결 시도 시 datanode에 값이 없다고 한다.

단계 별로 문제가 무엇인지 어떻게 해결할지를 구성해보았습니다.

결론 : ec2 인스턴스의 vpc 설정 값을 NameNode는 Private으로 DataNode는 Public으로 hosts를 구성하여 해결하였다.

2023-10-10, 2023-10-11 추가 업데이트

<br>

##### 1. 파일 생성 권한 사라지는 문제
---
- hadoop cluster 재시작 후 파일 생성 시 권한이 없다.
- hdfs 명령어로 모든 유저가 변경 가능하도록 권한 부여(특정 파일)

<a href="https://ibb.co/hfb77ky"><img src="https://i.ibb.co/Hd6FF0V/Untitled-43.png" alt="Untitled-43" border="0"></a>

```bash
$ hdfs dfs -chmod 777 /
```

##### 2. Spark에서 Hadoop으로 연결
---
- a) datanode 연결 설정 추가

```bash
$ sudo vi $HADOOP_HOME/etc/hadoop/core-site.xml

# 추가
<property>
    <name>dfs.client.use.datanode.hostname</name>
    <value>true</value>
</property>
```

- b) `Connection TimeOut`
- 일반적으로 대부분의 사람이 같은 Node에서 구성하였는지 연결 시 hdfs://localhost:port으로 시도하였으나, 현재 상황에서는 알맞는 답이 아니다.
- 하지만, 나의 상황에서는 별도의 Node에서 구성
- url : `hdfs://IP:Port`
- path : `/test/txt_dir/`

<a href="https://ibb.co/zZ4STCH"><img src="https://i.ibb.co/sqVFdcJ/Untitled-44.png" alt="Untitled-44" border="0"></a>


```bash
from pyspark import SparkConf, SparkContext

conf = SparkConf().setAppName("ReadHDFS")
sc = SparkContext(conf=conf)

hdfs_server = "hdfs://3.34.179.116:9000"
hdfs_file_path = "/test/123/"  # 사용자 이름을 본인의 HDFS 사용자 이름으로 대체하세요.

data = sc.textFile(hdfs_server+hdfs_file_path+"sample.txt")
first_lines = data.take(5)
for line in first_lines:
    print(line)
```

##### 3. 연결이 된 이후에 데이터를 가져올 때 Block 정보를 아는 DataNode가 없다고 한다.
---
- `Connection Timeout`
- 데이터 요청 시 Spark → NameNode → DataNode → 직접 연결할 것으로 예상
- 하지만, Hadoop Cluster 구성 시 private-ip로 서로를 바라본다. (동일 VPC에 존재하는 EC2 인스턴스이기 때문에)
- 그렇기에 NameNode에서 DataNode 값을 전달해줄 때, private-ip를 전달해주어 연결이 불가하여 connection 문제가 발생하는거 같은데.. 이 부분에 대해서는 조금 더 찾아봐야 겠다.
- DataNode를 Public으로 구성하는 경우 인스턴스 시작 시 binding 에러로 구성이 불가하다.

<a href="https://ibb.co/Xk80QqQ"><img src="https://i.ibb.co/PzQXLdL/2023-10-10-11-00-49.png" alt="2023-10-10-11-00-49" border="0"></a>

```bash
$ nc -v ip port
```

##### 3-1) ssh 연결 문제
---
- Hadoop [start-all.sh](http://start-all.sh) 구성 시 각 노드에 제대로 연결하지 못하는 이슈 발생
- ssh keygen 다시 설정하여 준다.

<a href="https://ibb.co/xC78W2C"><img src="https://i.ibb.co/p0Lrc10/2023-10-11-11-53-36.png" alt="2023-10-11-11-53-36" border="0"></a>

```bash
$ ssh-keygen -R worker01
```


##### 3-2) Node 별 Public&Private 구성 문제
---
- `http://Masterip:9870` Master Node의 Hadoop UI를 들어가보면, Worker Node들이 Private IP로 구성된 것을 볼 수 있다. 이 부분이 문제인 것일까?
- 찾아보니 NameNode, Secondary NameNode의 노드에서는 자신을 가리키는 /etc/hosts에서는 private-ip로 설정해야 한다. ( 그렇지 않으면 NameNode가 실행되지 않는다. )
- 이외의 DataNode에서는 무조건 Public으로 해야 외부에서 접근이 가능하며, 그렇지 않은 경우에는 접속이 불가능하다.
- 그렇다 이 방법이 해결책이였다. 결국, EC2 Instance의 Public Private을 각 노드 별 설정환경에 맞추어 값을 넣어주고 사용하면 된다.

<a href="https://ibb.co/vsFsZCY"><img src="https://i.ibb.co/pW7WJ6y/2023-10-11-11-03-22.png" alt="2023-10-11-11-03-22" border="0"></a>

<br>

##### 4) Hadoop Read&Write 처리하기
---
- Pyspark로 Hadoop의 있는 데이터를 Read하거나 Write하는 방법을 임시 Class로 구성하여 보았다.
- ConnectHadoop 클래스 안에 read_file과 write_file로 구성하였다.

```bash
from pyspark.sql import SparkSession
from datetime import datetime

class ConnectHadoop:
    def __init__(self, ip="masterip", port="9000", file_path="", file_name="", user_name="kim"):
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

    def write_file(self, df, write_type):
        if write_type == "parquet":
            df.write.format("parquet").mode("overwrite").save(self.hdfs_file_path)

if __name__ == "__main__":
    current_date = datetime.now().strftime("%Y-%m-%d")

    # read text file
    hadoop = ConnectHadoop(file_path="/test/", file_name="hadooptest.txt")
    data = hadoop.read_file(read_type="txt")
    print(data.take(5))

    # write parquet
    hadoop = ConnectHadoop(file_path=f"/test/log_dir/{current_date}")
    raw_data = [("Alice", 34), ("Bob", 45), ("Charlie", 29)]
    columns = ["Name", "Age"]
    df = hadoop.spark.createDataFrame(raw_data, columns)
    hadoop.write_file(df, write_type="parquet")

    # read parquet
    data = hadoop.read_file(read_type="parquet")
    print(data.take(5))
```

<a href="https://ibb.co/CWg0vKx"><img src="https://i.ibb.co/b7TdL5p/2023-10-11-11-08-54.png" alt="2023-10-11-11-08-54" border="0"></a>
<a href="https://ibb.co/KyC6mB7"><img src="https://i.ibb.co/dKnB7X6/2023-10-11-11-09-44.png" alt="2023-10-11-11-09-44" border="0"></a>

<br>

##### 참고
---
- https://engineering.linecorp.com/ko/blog/data-engineering-software-troubleshooting