---
layout: post
title:  "Airflow External"
author: Kimuksung
categories: [ Airflow ]
tags: [ Airflow, External ]
image: assets/images/airflow.png
# 댓글 기능
comments: False
---

안녕하세요
오늘은 Airflow External 개념에 대해 알아보겠습니다.


##### [External](https://tommybebe.github.io/2020/11/30/airflow-external-task-sensor/)

---

- `Dag B` 입장에서 다른 `Dag A`가 끝나면 실행되도록 하고 싶은 경우
- Dag A
    - Dag A는 별도로 추가할 코드 X
- Dag B
    - `ExternalTaskSensor` 사용 - [참고](https://airflow.apache.org/docs/apache-airflow/2.2.2/_api/airflow/sensors/external_task/index.html)

![https://ifh.cc/g/tcS7F2.png](https://ifh.cc/g/tcS7F2.png)

```python
#dag A
from datetime import datetime
from airflow import DAG
from airflow.operators.python_operator import PythonOperator

def print_execution_date(dt):
    print(dt)

default_args = {
    'owner': 'kai',
    'depends_on_past': False,
    'start_date': datetime(2023,1,26),
}

ds = ''
ts = ''

with DAG(
    dag_id='DAG_A', 
    schedule_interval='0 0 * * *',
    default_args=default_args) as dag:

    task_1 = PythonOperator(
        dag=dag,
        task_id='Task_1',
        python_callable=print_execution_date,
        op_kwargs={'ts': ts},
    )
    task_2 = PythonOperator(
        dag=dag,
        task_id='Task_2',
        python_callable=print_execution_date,
        op_kwargs={'ds': ds},
    )
    task_1 >> task_2
```

```python
from datetime import datetime, timedelta

from airflow import DAG
from airflow.sensors.external_task import ExternalTaskSensor
from airflow.operators.python_operator import PythonOperator

def print_execution_date(ds):
    print(ds)

ds = ''
start_date = datetime(2023, 1, 26)

default_args = {
    'owner': 'airflow',
    'depends_on_past': False,
    'start_date': start_date
}

with DAG(
    dag_id='DAG_B', 
    schedule_interval='0 0 * * *',
    default_args=default_args) as dag:

    sensor = ExternalTaskSensor(
        task_id='wait_for_task_2',
        external_dag_id='DAG_A',
        external_task_id='Task_2',
        start_date=start_date,
        execution_date_fn=lambda x: x,
        mode='reschedule',
        timeout=3600,
    )

    task_3 = PythonOperator(
        dag=dag,
        task_id='Task_3',
        python_callable=print_execution_date,
        op_kwargs={'ds': ds},
    )

    sensor >> task_3
```