---
layout: post
title:  "Airflow Sensor"
author: Kimuksung
categories: [ Airflow ]
tags: [ Airflow, Sensor ]
image: assets/images/airflow.png
# 댓글 기능
comments: False
---

안녕하세요  
오늘은 Airflow 스케줄링하며, 특정 주기 동안 조건을 만족했을 때 진행하도록 하기 위함  
데이터 마트 구축 시 Collection 별로 스케줄링 하는 것이 아닌 순차적으로 진행시키게 하도록 할 때(메모리 이슈로 분할 처리)

<br>

##### 정의
---
- 특정 주기 동안 조건을 만족한지 확인하는 오퍼레이터
- BaseSensorOperator를 상속
- 연속성을 가져 작업을 한번에 처리하는 경우

E.g) 새벽 Datamart 작업의 경우 계속하여 진행 처리하면 된다. 굳이 시간을 5분마다로 두어 동작할 이유가 없다.

<br>

##### Sensor
---
- FileSensor
    - 파일이 존재하는지 여부 체크
- PythonSensor
    - Python_callable func 요청 후 True 응답값을 받을 때 까지 대기
- ExternalTaskSensor
    - Task가 완료 까지 대기
- HttpSensor
    - API request가 성공 까지 대기
- SqlSensor
    - Sql 테이블 존재 까지 대기

<br>

##### Parameter
---
- `mode`
    - Sensor 동작 방식
    - poke
        - Default 방식
        - Sensor 전체 실행동안 worker 슬롯 점유
        - poke 사이에는 sleep 상태 존재
    - reschedule
        - 조건을 만족하지 않았을 때 worker 슬롯 해제
        - 긴 실행시간을 가질 것으로 예상될 때 주로 사용
        - 다른 테스크 실행할 수 있게 해주며, Resource 적게 소모
- `poke_interval`
    - poke 방식 사용 시 조건 확인하는 주기
    - Default = 30
- `exponential_backoff`
    - True 인 경우 poke 사이의 간격을 지수로 증가
- `timeout`
    - Task 확인할 최대 시간
    - 최대시간을 넘으면 실패
- `soft_fail`
    - True인 경우 Timeout이 나더라도 실패가 아닌 스킵
    
<br>

##### 결과 및 코드
---
![https://ifh.cc/g/dygAS9.png](https://ifh.cc/g/dygAS9.png)
![https://ifh.cc/g/rw4xnd.png](https://ifh.cc/g/rw4xnd.png)

```python
# DagA.py
from datetime import datetime
from airflow import DAG
from airflow.operators.python_operator import PythonOperator

def print_execution_date(**kwargs):
    print(kwargs)

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
#DagB.py
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