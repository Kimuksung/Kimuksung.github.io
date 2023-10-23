---
layout: post
title:  "Airflow Operator"
author: Kimuksung
categories: [ Airflow ]
tags: [ Airflow, Operator ]
image: assets/images/airflow.png
# 댓글 기능
comments: False
---

Airflow에서는 하나의 Workflow를 DAG로 구성한다.

DAG는 여러 개의 TASK로 구성된다.

TASK의 종류에는 Operator, Sensor, Hook이 존재한다.

오늘은 Operator에 대해 작성해볼 예정이다.

Operator는 다양하게 존재하니 필요에 따라 가져다 쓰면 된다.
- [DummyOperator 상세 링크](https://kimuksung.github.io/airflow-dummyoperator/)
- [BashOperator 상세 링크](https://kimuksung.github.io/airflow-bashoperator/)
- [PythonOperator 상세 링크](https://kimuksung.github.io/airflow-pythonoperator/)
- [SparkOperator 상세 링크](https://kimuksung.github.io/airflow-sparkoperator/)
- SimpleHttpOperator
- MySqlOperator
- PostgresOperator
- MsSqlOperator
- OracleOperator
- JdbcOperator
- DockerOperator
- HiveOperator
- S3FileTransformOperator
- PrestoToMySqlOperator
- SlackAPIOperator


참조
---
https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/operators.html