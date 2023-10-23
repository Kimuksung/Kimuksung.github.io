---
layout: post
title:  "Airflow DummyOperator"
author: Kimuksung
categories: [ Airflow ]
tags: [ Airflow, DummyOperator ]
image: assets/images/airflow.png
# 댓글 기능
comments: False
hidden: True
---

##### Dummy Operator
---
<aside>
💡 아무 작업을 하지 않는 Operator
</aside>

시작/끝/그룹화 시 사용한다.

의존적인 여러 TASK들을 구성하는 경우에 유용하다.

```python
from airflow.operators.dummy import DummyOperator
```

```python
default_args = {
    'owner': 'airflow',
    'depends_on_past': False,
    'start_date': datetime(2023, 10, 23),
    'email_on_failure': False,
    'email_on_retry': False,
    'retries': 1
}

dag = DAG('dummy operator test', default_args=default_args, schedule_interval='@daily')

start_task = DummyOperator(task_id='start_task', dag=dag)

task_1 = DummyOperator(task_id='task_1', dag=dag)
task_2 = DummyOperator(task_id='task_2', dag=dag)

end_task = DummyOperator(task_id='end_task', dag=dag)

start_task >> task_1 >> task_2 >> end_task
```

```python
from airflow.utils.task_group import TaskGroup

dag = DAG('dummy_operator_example', default_args=default_args, schedule_interval='@daily')

start_task = DummyOperator(task_id='start_task', dag=dag)

with TaskGroup(group_id="taskgroup", tooltip="Task group test") as group:
    task_1 = DummyOperator(task_id='task_1', dag=dag)
    task_2 = DummyOperator(task_id='task_2', dag=dag)
    task_3 = DummyOperator(task_id='task_2', dag=dag)

    task1 >> [task2, task3]

end_task = DummyOperator(task_id='end_task', dag=dag)

start_task >> taskgroup >> end_task
```

##### 참조
---
- https://medium.com/@agusmahari/getting-started-with-the-dummyoperator-in-airflow-simplifying-workflow-design-cd68048ff211