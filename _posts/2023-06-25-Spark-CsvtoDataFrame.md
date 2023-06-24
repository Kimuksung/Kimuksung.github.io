---
layout: post
title:  "Spark Csv -> DataFrame"
author: Kimuksung
categories: [ Spark ]
#blog post image
image: assets/images/emr_spark.png
comments: False
---


RDD에 이어 더 간편하게 사용가능한 Sparksession의 Dataframe 기능을 사용해보려고 합니다.

### 1. Read CSV

---

- sparksession에서 지원하여 주는 dataframe을 사용
- 데이터를 바라보고 타입을 유추
- `Schema Option` = 원하는 데이터 Schema가 있다면 세팅이 가능하다.
- 각 타입별로 import가 필요하며, Schema 구성 시 Struct, StructField가 필수적이다.
    
    ```python
    from pyspark.sql import SparkSession
    
    spark = SparkSession.\
            builder.\
            appName("baro-mysql-test").\
            master("spark://spark-master:7077").\
            config("spark.jars", "jars/mysql-connector-java-8.0.27.jar").\
            getOrCreate()
    
    file_dir = "datas/"
    file_name = "baro_dev_payment.csv"
    
    df = spark.read.csv(file_dir+file_name, header=True)
    df.show(5)
    df.printSchema()
    ```
    
    ```bash
    from pyspark.sql.types import StructType, StructField
    from pyspark.sql.types import DoubleType, IntegerType, StringType, TimestampType
    
    schema = StructType([
        StructField("id", IntegerType()),\
        StructField("created_at", TimestampType()),\
        StructField("updated_at", TimestampType()),\
        StructField("deleted_at", TimestampType()),\
        StructField("money_paid", IntegerType()),\
        StructField("pay_method", StringType()),\
        StructField("result", StringType()),\
        StructField("card_id", StringType()),\
        StructField("usage_id", StringType())
    ])
    
    file_dir = "datas/"
    file_name = "payment-test.csv"
    
    df = spark.read.csv(file_dir+file_name, header=True, schema=schema)
    df.show(5)
    df.printSchema()
    ```
    

### 2. mapping

---

- Dataframe은 별도의 Map 기능이 존재하지 않는다.
- 대신, 기존 데이터를 변경가능한 `withColumn`과 `col`을 통해 데이터를 mapping 가능하다.
- 특정 칼럼만 추출하고 싶은 경우 `select`를 사용한다.
- 아래는 KTC로 변경, 결제금액을 map한 코드이다.
    
    ```python
    # 결제 금액+1 / UTC->KTC
    from pyspark.sql.functions import col, from_utc_timestamp, date_format
    
    # dataframe은 map 함수 X
    map_df = df.withColumn("change_money_paid", col("money_paid") + 1)\
        .withColumn("change_ktc_created_at", from_utc_timestamp(col("created_at"),"Asia/Seoul"))\
        .withColumn("change_yyyymm_created_at", date_format(col("created_at"),"yyyy-MM"))\
        .select("id", "created_at","money_paid", "change_money_paid", "change_ktc_created_at", "change_yyyymm_created_at")
    
    map_df.show(5)
    ```