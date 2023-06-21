---
layout: post
title:  "Spark-Aws RDS 연결하기"
author: Kimuksung
categories: [ Spark, AWS, RDS ]
#blog post image
image: assets/images/emr_spark.png
comments: False
---

AWS RDS Mysql과 Spark를 연결하는 과정에서 많은 애로사항을 겪었고 이를 간략하게 정리하여 봅니다.

ssh를 통한 연결 방법은 추가적으로 업데이트 할 예정입니다.
* 업데이트 내용 : 6/20 - jdbc 파일 연결 방법 추가
필수적인 순서는 다음과 같습니다.
1. JDBC 다운
2. Create Session
3. driver 설정 및 Database 연결

<br>

##### JBDC 다운
---
- Database 연결에 적합한 JDBC jar파일을 받아줍니다. - [링크](https://github.com/awslabs/aws-mysql-jdbc/releases/download/1.0.0/aws-mysql-jdbc-1.0.0.jar)
    
    ```python
    import wget
    
    url = "https://github.com/awslabs/aws-mysql-jdbc/releases/download/1.0.0/aws-mysql-jdbc-1.0.0.jar"
    wget.download(url)
    ```
    
- Spark Cluster들이 Mount되어 있는 파일에 이를 연결하여 줍니다. 혹은 Spark Cluster Node들이 해당 경로에 제대로된 jars 파일 확인
- Error : `Py4JJavaError: An error occurred while calling o163.load.: java.lang.ClassNotFoundException: com.mysql.jdbc.Driver`

- 이 때 Mount가 되지 않는다면, **`Docker Version이 달라`** 발생할 수 있습니다. ( 저의 경우는 docker-compose 2.12 버전인데 3을 사용하고 있어 Mount처리가 되지 않음 )
    
    ```bash
    $ docker exec -it container_id /bin/bash
    $ cd mount_file_directory
    ```
    

##### Create Session
---
- Session 연결 시 Driver를 연결해야주어야 하는데 아래 2가지가 존재하는 것으로 이해하였습니다.
- spark.jars = 드라이버 파일을 명시적으로 로드, Cluster Node에 존재하는 jar파일 경로를 넣어 연동
- *추가 - 쥬피터에서 jar 파일을 찾을 때에는 python의 현재위치부터 주어져야합니다.
- 현재 jar 파일은 - opt/workspace/spark_test/jars에 들어있습니다. [참조] - (https://stackoverflow.com/questions/2983248/com-mysql-jdbc-exceptions-jdbc4-communicationsexception-communications-link-fai)

    ```python
    from pyspark.sql import SparkSession
    
    # Create SparkSession
    spark = SparkSession.\
        builder.\
        appName("pyspark-notebook").\
        master("spark://spark-master:7077").\
        config("spark.jars", "jars/mysql-connector-java-8.0.27.jar").\
        getOrCreate()
    ```
    
- spark.driver.extraClassPath
    - 외부(S3,등)에 존재하는 jar파일 경로를 넣어 연동
    - 모든 워커 노드에 수동으로 배포 필수
    
    ```python
    from pyspark.sql import SparkSession
    
    # Create SparkSession
    spark = SparkSession \
        .builder.config("spark.driver.extraClassPath", "s3://bucket-name/file-directory/connector.jar") \
        .master("local") \
        .appName("PySpark_MySQL_test") \
        .getOrCreate()
    ```
    

##### Driver 설정 및 Database 연결
---
- Database 정보값은 별도로 구성합니다.
- Driver 값은 Database 정보에 맞는 값을 넣어주어야 합니다.
- driver - aws rds mysql이기에 mysql,mysql-rdbs 값 둘다 연결됩니다.
    - mysql-rdbs = **`com.mysql.cj.jdbc.Driver`**
    - maria = **`org.mariadb.jdbc.Driver`**
    - mysql = **`com.mysql.jdbc.Driver`**
- error :`requirement failed: The driver could not open a JDBC connection.`
- Database에 맞지 않는 driver를 사용

```python
sql_url = "aws-rds-endpoint"
user = "db_id"
password = "db_pw"
database = "database"
tables = "talbe-name"
```

```python
df = spark.read \
    .format("jdbc") \
    .option("driver", "com.mysql.cj.jdbc.Driver") \
    .option("url", f"jdbc:mysql://{sql_url}:{port}/{database}") \
    .option("dbtable", tables) \
    .option("user", user) \
    .option("password", password) \
    .load()

df.printSchema()
```

![result](https://i.ibb.co/ZzxrLrB/2023-06-17-8-57-30.png)