---
layout: post
title:  "Airflow SparkOperator"
author: Kimuksung
categories: [ Airflow ]
tags: [ Airflow, SparkOperator ]
image: assets/images/airflow.png
# 댓글 기능
comments: False
hidden: True
---

##### SparkOperator
---
```python
$ pip install pyspark
```

Python Operator로 구성하기

```python
import airflow
from datetime import datetime, timedelta
from airflow.operators.python_operator import PythonOperator
from pyspark.sql import SparkSession, functions

def process_etl_spark():
    spark = SparkSession \
        .builder \
        .appName("Extração Documentos a Pagar") \
        .config("spark.jars.packages",
                "org.mongodb.spark:mongo-spark-connector_2.12:3.0.0,com.microsoft.sqlserver:mssql-jdbc:8.4.1.jre8") \
        .config("spark.mongodb.input.uri", "mongodb://127.0.0.1:27017/Financeiro") \
        .config("spark.mongodb.output.uri", "mongodb://127.0.0.1:27017/Financeiro") \
        .config("spark.driver.maxResultSize", "8g") \
        .config("spark.network.timeout", 10000000) \
        .config("spark.executor.heartbeatInterval", 10000000) \
        .config("spark.storage.blockManagerSlaveTimeoutMs", 10000000) \
        .config("spark.executor.memory", "10g") \
        .master("spark://192.168.0.1:7077") \
        .getOrCreate()

    start_time = datetime.now()

    df = spark.read.format("jdbc") \
        .option("url", "jdbc:sqlserver://127.0.0.1:1433;databaseName=Teste") \
        .option("user", 'Teste') \
        .option("password", 'teste') \
        .option("numPartitions", 100) \
        .option("partitionColumn", "Id") \
        .option("lowerBound", 1) \
        .option("upperBound", 488777675) \
        .option("driver", "com.microsoft.sqlserver.jdbc.SQLServerDriver") \
        .option("dbtable", "(select Id, DataVencimento AS Vencimento, TipoCod AS CodigoTipoDocumento, cast(recsld as FLOAT) AS Saldo from DocumentoPagar \
         where TipoCod in ('200','17') and RecPag = 'A') T") \
        .load()
    
    group = df.select("CodigoTipoDocumento", "Vencimento", "Saldo") \
        .groupby(["CodigoTipoDocumento", "Vencimento"]).agg(functions.sum("Saldo").alias("Saldo"))

    group.write.format("com.mongodb.spark.sql.DefaultSource") \
        .mode("overwrite") \
        .option("database", "Financeiro") \
        .option("collection", "Fact_DocumentoPagar") \
        .save()

    end_time = datetime.now()

    print(start_time)
    print(end_time - start_time )

default_args = {
    'owner': 'airflow',
    'start_date': datetime(2023, 10, 23),
    'retries': 10
}

start_task = DummyOperator(task_id='start_task', dag=dag)

end_task = DummyOperator(task_id='end_task', dag=dag)

task1 = PythonOperator(
    task_id= "pyspark_example",
    python_callable= process_etl_spark
    dag=dag
)

dag = DAG('spark with python operator', default_args=default_args, schedule_interval='@daily')

start_task >> task1 >> end_task 
```

##### 참조
---
- https://medium.com/codex/executing-spark-jobs-with-apache-airflow-3596717bbbe3