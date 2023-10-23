---
layout: post
title:  "Airflow PythonOperator"
author: Kimuksung
categories: [ Airflow ]
tags: [ Airflow, PythonOperator ]
image: assets/images/airflow.png
# 댓글 기능
comments: False
hidden: True
---

##### Python Operator
---
<aside>
💡 Python 코드를 실행하는 Operator
</aside>

```python
classairflow.operators.python.PythonOperator(
    *, 
    python_callable: Callable, 
    op_args: Optional[List] = None, 
    op_kwargs: Optional[Dict] = None, 
    templates_dict: Optional[Dict] = None, 
    templates_exts: Optional[List[str]] = None, 
    **kwargs
)
```

- op_kwargs : dictionary argument
- op_args : list argument

```python
from airflow.operators.python import PythonOperator
```

```python
default_args = {
    'owner': 'airflow',
    'start_date': datetime(2023, 10, 23)
}

dag = DAG('python_operator', default_args=default_args, schedule_interval='@daily')

start_task = DummyOperator(task_id='start_task', dag=dag)

end_task = DummyOperator(task_id='end_task', dag=dag)

def test_func():
    print(f'Test function Executed')

task1 = PythonOperator(
    task_id= "task1",
    python_callable= test_function
    op_kwargs= { "key" : "value"},
    dag=dag
)

start_task >> task_1 >> end_task
```

##### 참조
---
https://avinashnavlani.medium.com/airflow-operators-161f16102403