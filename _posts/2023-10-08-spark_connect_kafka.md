---
layout: post
title:  "Spark - Kafka 연동하기"
author: Kimuksung
categories: [ Spark ]
#blog post image
image: assets/images/emr_spark.png
comments: False
---

Aws Instance로 Kafka클러스터를 구성한 뒤, Docker 위에 Spark를 구성하여 Hadoop으로 전송하려고 합니다.

구성 중에 Spark와 Kafka 연동이 되지 않아 해결 방안을 찾고 있습니다.

- jar 파일로 구성 - 시도(실패) [링크](https://mvnrepository.com/artifact/org.apache.spark/spark-sql-kafka-0-10_2.12/3.1.1)
- pyspark —packages로 구성 - 시도(실패) [링크](https://saturncloud.io/blog/troubleshooting-pysparksqlutilsanalysisexception-failed-to-find-data-source-kafka/)
- spark config jupyter로 구성 - 시도(실패)
- dependency 추가(이해 못함) - [링크](https://stackoverflow.com/questions/48011941/why-does-formatkafka-fail-with-failed-to-find-data-source-kafka-even-wi)

스파크 버전은 3.1.1이며, scala 버전 = 2.12, Kafka = 3.0.0 입니다.

시도했던 과정들을 다시 적어보며, 다시 도전해볼 예정입니다.

##### Jar 파일 구성
---
- [링크](https://mvnrepository.com/artifact/org.apache.spark/spark-sql-kafka-0-10_2.12/3.1.1)에서 본인의 버전에 맞게 다운로드하여 줍니다. version이 sparkversion을 의미합니다.
    - `spark-sql-kafka-0-10_2.12-3.1.1.jar`
- Jupyter에 Jar 파일 구성
    
    ```bash
    from pyspark.sql import SparkSession
    
    # spark version - '3.1.1'
    # kafka version - '3.0.0'
    
    sc = SparkSession.\
             builder.\
             appName("Kafka-test").\
             master("spark://spark-master:7077").\
             config("spark.jars.pacakges", "jars/spark-sql-kafka-0-10_2.12-3.1.1.jar").\
             getOrCreate()
    
    sc.sparkContext.setLogLevel('ERROR')
    
    # Subscribe to 1 topic
    df = sc\
      .readStream \
      .format("kafka") \
      .option("kafka.bootstrap.servers", "kafkaip:9092") \
      .option("subscribe", "test") \
      .load()
    
    df.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")
    ```
    
- SPARK_HOME/jars 에 구성
    
    ```bash
    $ cd $SPARK_HOME/jars
    $ wget https://repo1.maven.org/maven2/org/apache/spark/spark-sql-kafka-0-10_2.12/3.1.1/spark-sql-kafka-0-10_2.12-3.1.1.jar
    ```
    
    <a href="https://ibb.co/X7JWhMJ"><img src="https://i.ibb.co/gSgr5hg/Untitled-38.png" alt="Untitled-38" border="0"></a>
    

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

##### 참조
---
- https://spark.apache.org/docs/latest/structured-streaming-kafka-integration.html
- https://taaewoo.tistory.com/78?category=926385
- https://data-engineer-tech.tistory.com/46
- https://velog.io/@statco19/pyspark-kafka-streaming