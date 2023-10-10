---
layout: post
title:  "Spark - Kafka 연동하기"
author: Kimuksung
categories: [ Spark ]
#blog post image
image: assets/images/emr_spark.png
comments: False
featured: true
---

Aws Instance로 Kafka클러스터를 구성한 뒤, Docker 위에 Spark를 구성하여 Hadoop으로 전송하려고 합니다.

구성 중에 Spark와 Kafka 연동이 되지 않아 해결 방안을 찾고 있습니다.

- jar 파일로 구성 - 시도(실패) [링크](https://mvnrepository.com/artifact/org.apache.spark/spark-sql-kafka-0-10_2.12/3.1.1)
- pyspark —packages로 구성 - 시도(실패) [링크](https://saturncloud.io/blog/troubleshooting-pysparksqlutilsanalysisexception-failed-to-find-data-source-kafka/)
- spark config jupyter로 구성 - 시도(**성공**)
- dependency 추가(이해 못함) - [링크](https://stackoverflow.com/questions/48011941/why-does-formatkafka-fail-with-failed-to-find-data-source-kafka-even-wi)

스파크 버전은 3.1.1이며, scala 버전 = 2.12, Kafka = 3.0.0 입니다.

Jupyter에서 pyspark로 Kafka 연결하는 방법입니다.

결론은 conf에 **spark.jars.packages로 구성하여 연동은 가능**하나, local에 저장하여 구성하는 spark.jars로는 실패하였습니다. 차후에 시간이 되면 해당 부분 업데이트 하겠습니다.

##### Jar 파일 구성
---
- [링크](https://mvnrepository.com/artifact/org.apache.spark/spark-sql-kafka-0-10_2.12/3.1.1)에서 본인의 버전에 맞게 다운로드하여 줍니다. version이 sparkversion을 의미합니다.
    - `spark-sql-kafka-0-10_2.12-3.1.1.jar`
    - `kafka-clients 3.2.0.jar`
    - `spark-token-provider-kafka-0-10_2.12.3.1.1.jar`
    - `commons_pools:2.6.2.jar`
- Jupyter에 Jar 파일 구성
    - stackoverflow를 보면 jars 파일로는 dependecncy로 불가하다는데,, 이유를 모르겠습니다.
    - packages에서 성공한 jars를 가지고 구성해보았습니다.
    - https://stackoverflow.com/questions/46076771/how-to-submit-multiple-jars-to-workers-through-sparksession/46076881#46076881
        
        ```bash
        from pyspark.sql import SparkSession
            
        jars = [
                "kafka-clients-3.2.0.jar", 
                "spark-sql-kafka-0-10_2.12-3.1.1.jar"
               ]
        jar_files = ",".join(list(map(lambda x:"/opt/workspace/spark_test/jars/kafka/"+x, jars)))
        
        spark = SparkSession.builder\
           .master("local")\
           .appName("Kafka Connection test")\
           .config("spark.jars", jar_files)\
           .getOrCreate()
        
        spark
        ```
        
        ```bash
        # upload된 파일 확인하기
        # https://stackoverflow.com/questions/57057648/list-all-additional-jars-loaded-in-pyspark
        print(spark.sparkContext._jsc.sc().listJars())
        ```
        
- Spark Node
    - SPARK_HOME/jars 다운로드
    
    ```bash
    $ cd $SPARK_HOME/jars
    $ wget https://repo1.maven.org/maven2/org/apache/spark/spark-sql-kafka-0-10_2.12/3.1.1/spark-sql-kafka-0-10_2.12-3.1.1.jar
    ```
    
    <a href="https://ibb.co/X7JWhMJ"><img src="https://i.ibb.co/gSgr5hg/Untitled-38.png" alt="Untitled-38" border="0"></a>
    
    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/841a625d-aa66-4b40-81fe-31e8b5793fa0/85819fa9-069b-413a-9f04-8d3d112f8dbd/Untitled.png)
    

##### Pyspark option 주기
---
- docker에 직접 접속하여 command option으로 날려보았습니다.

```bash
$ docker -exec -it spark-master /bin/bash
$ docker exec -it spark-master /bin/bash
$ pyspark --packages org.apache.spark:spark-sql-kafka-0-10_2.12:3.1.2

spark.sparkContext.getConf().getAll()
> [('spark.app.id', 'local-1696780510280'), ('spark.app.startTime', '1696780509674'), ('spark.submit.pyFiles', '/root/.ivy2/jars/org.apache.spark_spark-sql-kafka-0-10_2.12-3.1.2.jar,/root/.ivy2/jars/org.apache.spark_spark-token-provider-kafka-0-10_2.12-3.1.2.jar,/root/.ivy2/jars/org.apache.kafka_kafka-clients-2.6.0.jar,/root/.ivy2/jars/org.apache.commons_commons-pool2-2.6.2.jar,/root/.ivy2/jars/org.spark-project.spark_unused-1.0.0.jar,/root/.ivy2/jars/com.github.luben_zstd-jni-1.4.8-1.jar,/root/.ivy2/jars/org.lz4_lz4-java-1.7.1.jar,/root/.ivy2/jars/org.xerial.snappy_snappy-java-1.1.8.2.jar,/root/.ivy2/jars/org.slf4j_slf4j-api-1.7.30.jar'), ('spark.app.initial.file.urls', 'file:///root/.ivy2/jars/com.github.luben_zstd-jni-1.4.8-1.jar,file:///root/.ivy2/jars/org.apache.commons_commons-pool2-2.6.2.jar,file:///root/.ivy2/jars/org.apache.kafka_kafka-clients-2.6.0.jar,file:///root/.ivy2/jars/org.spark-project.spark_unused-1.0.0.jar,file:///root/.ivy2/jars/org.lz4_lz4-java-1.7.1.jar,file:///root/.ivy2/jars/org.slf4j_slf4j-api-1.7.30.jar,file:///root/.ivy2/jars/org.xerial.snappy_snappy-java-1.1.8.2.jar,file:///root/.ivy2/jars/org.apache.spark_spark-token-provider-kafka-0-10_2.12-3.1.2.jar,file:///root/.ivy2/jars/org.apache.spark_spark-sql-kafka-0-10_2.12-3.1.2.jar'), ('spark.sql.warehouse.dir', 'file:/usr/bin/spark-3.1.1-bin-hadoop2.7/spark-warehouse'), ('spark.jars', 'file:///root/.ivy2/jars/org.apache.spark_spark-sql-kafka-0-10_2.12-3.1.2.jar,file:///root/.ivy2/jars/org.apache.spark_spark-token-provider-kafka-0-10_2.12-3.1.2.jar,file:///root/.ivy2/jars/org.apache.kafka_kafka-clients-2.6.0.jar,file:///root/.ivy2/jars/org.apache.commons_commons-pool2-2.6.2.jar,file:///root/.ivy2/jars/org.spark-project.spark_unused-1.0.0.jar,file:///root/.ivy2/jars/com.github.luben_zstd-jni-1.4.8-1.jar,file:///root/.ivy2/jars/org.lz4_lz4-java-1.7.1.jar,file:///root/.ivy2/jars/org.xerial.snappy_snappy-java-1.1.8.2.jar,file:///root/.ivy2/jars/org.slf4j_slf4j-api-1.7.30.jar'), ('spark.driver.port', '41769'), ('spark.executor.id', 'driver'), ('spark.app.name', 'PySparkShell'), ('spark.app.initial.jar.urls', 'spark://8949e9a73a84:41769/jars/org.lz4_lz4-java-1.7.1.jar,spark://8949e9a73a84:41769/jars/org.apache.spark_spark-token-provider-kafka-0-10_2.12-3.1.2.jar,spark://8949e9a73a84:41769/jars/org.apache.spark_spark-sql-kafka-0-10_2.12-3.1.2.jar,spark://8949e9a73a84:41769/jars/org.apache.kafka_kafka-clients-2.6.0.jar,spark://8949e9a73a84:41769/jars/org.spark-project.spark_unused-1.0.0.jar,spark://8949e9a73a84:41769/jars/org.apache.commons_commons-pool2-2.6.2.jar,spark://8949e9a73a84:41769/jars/com.github.luben_zstd-jni-1.4.8-1.jar,spark://8949e9a73a84:41769/jars/org.slf4j_slf4j-api-1.7.30.jar,spark://8949e9a73a84:41769/jars/org.xerial.snappy_snappy-java-1.1.8.2.jar'), ('spark.sql.catalogImplementation', 'hive'), ('spark.rdd.compress', 'True'), ('spark.files', 'file:///root/.ivy2/jars/org.apache.spark_spark-sql-kafka-0-10_2.12-3.1.2.jar,file:///root/.ivy2/jars/org.apache.spark_spark-token-provider-kafka-0-10_2.12-3.1.2.jar,file:///root/.ivy2/jars/org.apache.kafka_kafka-clients-2.6.0.jar,file:///root/.ivy2/jars/org.apache.commons_commons-pool2-2.6.2.jar,file:///root/.ivy2/jars/org.spark-project.spark_unused-1.0.0.jar,file:///root/.ivy2/jars/com.github.luben_zstd-jni-1.4.8-1.jar,file:///root/.ivy2/jars/org.lz4_lz4-java-1.7.1.jar,file:///root/.ivy2/jars/org.xerial.snappy_snappy-java-1.1.8.2.jar,file:///root/.ivy2/jars/org.slf4j_slf4j-api-1.7.30.jar'), ('spark.serializer.objectStreamReset', '100'), ('spark.master', 'local[*]'), ('spark.repl.local.jars', 'file:///root/.ivy2/jars/org.apache.spark_spark-sql-kafka-0-10_2.12-3.1.2.jar,file:///root/.ivy2/jars/org.apache.spark_spark-token-provider-kafka-0-10_2.12-3.1.2.jar,file:///root/.ivy2/jars/org.apache.kafka_kafka-clients-2.6.0.jar,file:///root/.ivy2/jars/org.apache.commons_commons-pool2-2.6.2.jar,file:///root/.ivy2/jars/org.spark-project.spark_unused-1.0.0.jar,file:///root/.ivy2/jars/com.github.luben_zstd-jni-1.4.8-1.jar,file:///root/.ivy2/jars/org.lz4_lz4-java-1.7.1.jar,file:///root/.ivy2/jars/org.xerial.snappy_snappy-java-1.1.8.2.jar,file:///root/.ivy2/jars/org.slf4j_slf4j-api-1.7.30.jar'), ('spark.submit.deployMode', 'client'), ('spark.driver.host', '8949e9a73a84'), ('spark.ui.showConsoleProgress', 'true')]

df = spark\
  .readStream \
  .format("kafka") \
  .option("kafka.bootstrap.servers", ":9092") \
  .option("subscribe", "test") \
  .load()

Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/usr/bin/spark-3.1.1-bin-hadoop2.7/python/pyspark/sql/streaming.py", line 482, in load
    return self._df(self._jreader.load())
  File "/usr/local/lib/python3.9/dist-packages/py4j/java_gateway.py", line 1322, in __call__
    return_value = get_return_value(
  File "/usr/bin/spark-3.1.1-bin-hadoop2.7/python/pyspark/sql/utils.py", line 111, in deco
    return f(*a, **kw)
  File "/usr/local/lib/python3.9/dist-packages/py4j/protocol.py", line 326, in get_return_value
    raise Py4JJavaError(
py4j.protocol.Py4JJavaError: An error occurred while calling o160.load.
: java.lang.NoClassDefFoundError: org/apache/kafka/common/serialization/ByteArraySerializer
	at org.apache.spark.sql.kafka010.KafkaSourceProvider$.<init>(KafkaSourceProvider.scala:557)
	at org.apache.spark.sql.kafka010.KafkaSourceProvider$.<clinit>(KafkaSourceProvider.scala)
	at org.apache.spark.sql.kafka010.KafkaSourceProvider.org$apache$spark$sql$kafka010$KafkaSourceProvider$$validateStreamOptions(KafkaSourceProvider.scala:326)
	at org.apache.spark.sql.kafka010.KafkaSourceProvider.sourceSchema(KafkaSourceProvider.scala:71)
	at org.apache.spark.sql.execution.datasources.DataSource.sourceSchema(DataSource.scala:235)
	at org.apache.spark.sql.execution.datasources.DataSource.sourceInfo$lzycompute(DataSource.scala:116)
	at org.apache.spark.sql.execution.datasources.DataSource.sourceInfo(DataSource.scala:116)
	at org.apache.spark.sql.execution.streaming.StreamingRelation$.apply(StreamingRelation.scala:33)
	at org.apache.spark.sql.streaming.DataStreamReader.loadInternal(DataStreamReader.scala:220)
	at org.apache.spark.sql.streaming.DataStreamReader.load(DataStreamReader.scala:195)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at py4j.reflection.MethodInvoker.invoke(MethodInvoker.java:244)
	at py4j.reflection.ReflectionEngine.invoke(ReflectionEngine.java:357)
	at py4j.Gateway.invoke(Gateway.java:282)
	at py4j.commands.AbstractCommand.invokeMethod(AbstractCommand.java:132)
	at py4j.commands.CallCommand.execute(CallCommand.java:79)
	at py4j.GatewayConnection.run(GatewayConnection.java:238)
	at java.lang.Thread.run(Thread.java:750)
Caused by: java.lang.ClassNotFoundException: org.apache.kafka.common.serialization.ByteArraySerializer
	at java.net.URLClassLoader.findClass(URLClassLoader.java:387)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:418)
	at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:352)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:351)
	... 21 more
```

<a href="https://ibb.co/KGn9fcj"><img src="https://i.ibb.co/yR28Zmn/Untitled-39.png" alt="Untitled-39" border="0"></a>

##### Spark Conf로 구성하기
---

- conf의 spark.jars.packages에 요청하고자 하는 값을 넣어줍니다.
- https://github.com/OneCricketeer/docker-stacks/blob/master/hadoop-spark/spark-notebooks/kafka-sql.ipynb

```bash
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
spark
```

<a href="https://ibb.co/B3QqCr3"><img src="https://i.ibb.co/XLKSC3L/Untitled-40.png" alt="Untitled-40" border="0"></a>


```bash
# batch
# https://github.com/OneCricketeer/docker-stacks/blob/master/hadoop-spark/spark-notebooks/kafka-sql.ipynb
from pyspark.sql.functions import col, concat, lit

kafkaDf = spark.read.format("kafka")\
  .option("kafka.bootstrap.servers", "kafkaip:9092")\
  .option("subscribe", 'test')\
  .option("startingOffsets", "earliest")\
  .load()

kafkaDf.select(
    concat(col("topic"), lit(':'), col("partition").cast("string")).alias("topic_partition"),
    col("offset"),
    col("value").cast("string")
).show()
```

<a href="https://ibb.co/Rb4kTLc"><img src="https://i.ibb.co/NCnd2wK/Untitled-41.png" alt="Untitled-41" border="0"></a>


```bash
# stream
from pyspark.sql.functions import col, concat, lit
# Read stream
log = spark.readStream.format("kafka") \
.option("kafka.bootstrap.servers", "kafkaip:9092") \
.option("subscribe", "test") \
.option("startingOffsets", "earliest") \
.load()

query = log.selectExpr("CAST(value AS STRING)") \
.writeStream \
.format("console") \
.option("truncate", "false") \
.start()

query.awaitTermination()
```

<a href="https://ibb.co/Rb4kTLc"><img src="https://i.ibb.co/NCnd2wK/Untitled-41.png" alt="Untitled-41" border="0"></a>


##### 참조
---
- https://spark.apache.org/docs/latest/structured-streaming-kafka-integration.html
- https://taaewoo.tistory.com/78?category=926385
- https://data-engineer-tech.tistory.com/46
- https://velog.io/@statco19/pyspark-kafka-streaming
- https://mj-sunflower.tistory.com/52