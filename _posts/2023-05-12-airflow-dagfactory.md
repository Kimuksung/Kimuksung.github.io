---
layout: post
title:  "Airflow Dag Factory"
author: Kimuksung
categories: [ Airflow ]
tags: [ Airflow, Dag Factory ]
image: assets/images/airflow.png
# 댓글 기능
comments: False
---


##### Dag Factory란?

---
- 유사한 로직으로 동일하게 수행되는 Dag들을 한번에 관리

##### 사용 방법

---
`데이터 마트에` 적용 
- `유사한 로직으로 동일하게 수행`되는 Dag들을 한번에 관리하기 위한 장치
- 결국, `Dag → Class` 처럼 활용

Dag 관련 옵션 default_args,start_date 등을 공통 구성
- 수정할 때에도 빠르게 수정 가능하다.
- Python Operator를 활용하여서 간단하게 구축 가능
- BashOperator를 동적으로 활용
- 함수 자체를 넘겨주어 동적 돌아가도록 만든다.

기존 Dag Factory 라이브러리를 구성된 환경에 맞추어 재구성

- 기존 Dag Factory → Python Operator만 지원
- `BashOpertator` 도입
- `External` 사용하여 연속적으로 실행될 수 있도록

##### 결과
---
- 아래 코드를 보면 확연하게 줄어든것을 알 수 있다.
- 갱신된 코드

```sql
from dag_factory import DAGFactory

bash_collection = 'matchings'
DAG_NAME = 'dag_factory_datamart_matchings'

def task_Common_Code(context):
    import os
    collection,ts = context['collection'], context['ts']
    airflow_home = os.environ.get("AIRFLOW_HOME") 
    return (collection,ts,airflow_home)

def create(context):   
    collection,ts,airflow_home = task_Common_Code(context)
    return f'cd {airflow_home}/dags && python3 datamart/table_create.py datamart {collection} '

def delete(context):   
    collection,ts,airflow_home = task_Common_Code(context)
    return f'cd {airflow_home}/dags && python3 datamart/table_delete.py datamart {collection} '

def extract(context):
    collection,ts,airflow_home = task_Common_Code(context)
    return f'cd {airflow_home}/dags && python3 datamart/table_extract.py wclub {collection} {ts} '

def load_redshift(context):
    collection,ts,airflow_home = task_Common_Code(context)
    return f'cd {airflow_home}/dags && python3 datamart/table_load_redshift.py datamart {collection} {ts} '

def transform_redshift(context):
    collection,ts,airflow_home = task_Common_Code(context)
    return f'cd {airflow_home}/dags && python3 datamart/table_transform.py datamart {collection} '

tasks = {}
task_infos = [delete,create,extract,load_redshift,transform_redshift]
tasks[task_infos[0]] = []
for before_task,after_task in zip(task_infos,task_infos[1:]) :
    tasks[after_task] = [before_task]

dag = DAGFactory().get_airflow_dag(DAG_NAME, tasks , cron="55 17 * * *",collection=bash_collection)
```

- 갱신 전 코드

```sql
from airflow import DAG
from airflow.operators.bash_operator import BashOperator
from datetime import datetime , timedelta
from wclub.slackalert import SlackAlert

DAG_NAME = 'datamart_points'
DAG_DESCRIPTION = 'datamart_points_elt'
START_TIME = datetime(2023,1,17) 
SCHEDULER = "50 17 * * *"
SLACK = SlackAlert()

default_args = {
    'owner': 'kai' ,
    'on_failure_callback': SLACK.alert ,
    'retries': 2,
    'retry_delay': timedelta(minutes=30),
}

dag = DAG( 
    DAG_NAME,
    default_args = default_args,
    description=DAG_DESCRIPTION, 
    start_date=START_TIME, 
    schedule_interval = SCHEDULER 
)

bash_collection = 'points'
create = BashOperator(
    task_id='create',
    bash_command='cd ${AIRFLOW_HOME}/dags && python3 datamart/table_create.py datamart '+f'{bash_collection} ',
    dag=dag
)

delete = BashOperator(
    task_id='delete',
    bash_command='cd ${AIRFLOW_HOME}/dags && python3 datamart/table_delete.py datamart '+f'{bash_collection} ',
    dag=dag
)

extract = BashOperator(
    task_id='extract',
    bash_command='cd ${AIRFLOW_HOME}/dags && python3 datamart/table_extract.py wclub '+f'{bash_collection} ' +' {{ts}} ',
    dag=dag
)

load_redshift = BashOperator(
    task_id='load_redshift',
    bash_command='cd ${AIRFLOW_HOME}/dags && python3 datamart/table_load_redshift.py datamart '+ f'{bash_collection} ' +'{{ts}} ',
    dag=dag
)

transform_redshift = BashOperator(
    task_id='transform_redshift',
    bash_command='cd ${AIRFLOW_HOME}/dags && python3 datamart/table_transform.py datamart '+f'{bash_collection} ',
    dag=dag,
    on_success_callback=SLACK.success_alert
)

extract >> delete >> create >> load_redshift >> transform_redshift
```

```python
"""
This file holds the DAG Factory
"""

from airflow import DAG
from airflow.operators.bash_operator import BashOperator
from datetime import datetime, timedelta
from wclub.slackalert import SlackAlert
from airflow.sensors.external_task import ExternalTaskSensor
class DAGFactory:
    """
    Class that provides useful method to build an Airflow DAG
    """
    @classmethod
    def create_dag(cls, dagname, default_args={}, catchup=False, concurrency=5, cron=None):
        """
        params:
            dagname(str): the name of the dag
            default_args(dict): a dict with the specific keys you want to edit from the original DEFAULT_ARGS
            catchup(bool): Perform scheduler catchup (or only run latest)? Defaults to True
            concurrency(int): the number of task instances allowed to run concurrently
            cron(str): the cron expression or the schedule
        returns:
            DAG object
        """
        slack = SlackAlert()
        DEFAULT_ARGS = {
            'owner': 'Kai',
            'depends_on_past': False,
            'start_date': datetime(2023,1,18),
            'retries': 2,
            'retry_delay': timedelta(minutes=30),
            'on_failure_callback': slack.alert ,
            'on_success_callback':slack.success_alert
        }

        DEFAULT_ARGS.update(default_args)
        dagargs = {
            'default_args': DEFAULT_ARGS,
            'schedule_interval': cron,
            'catchup': catchup,
            'concurrency': concurrency
        }

        dag = DAG(dagname, **dagargs)
        return dag

    @classmethod
    def add_tasks_to_dag(cls, dag, tasks, collection):
        """
        Adds tasks to DAG object, sets upstream for each task.
        params:
            dag(DAG)
            tasks(dict): dictionary in which each key is a callback. The value of that key is the task's dependencies.
            If a task has no dependencies (it's the first task), set an empty list [] as the value.
            IMPORTANT: all tasks have to be there even if they don't have dependencies
        returns:
            dag(DAG) with tasks
        """
        with dag as dag:
            aux_dict = {}

            # create task objects and store them in a dictionary of "func name": task
            for func in tasks:
                task_id = func.__name__
                if task_id == 'external_dag_info':
                    before_dag = func()
                    print(before_dag)

                    task = ExternalTaskSensor(
                            task_id=f'wait_for_dag',
                            external_dag_id = before_dag['before_dag_id'],
                            external_task_id = before_dag['before_task'],
                            execution_date_fn = lambda x: x,
                            mode='reschedule',
                            timeout=3600,
                        )
                else :
                    task = BashOperator(
                        task_id=func.__name__,
                        bash_command=func( {'collection': collection ,'ts' : '{{ts}}' }),
                        dag=dag
                    )
                aux_dict[task_id] = task

            # for each task, set up the tasks predecessors
            for func, dependencies in tasks.items():
                task_id = func.__name__
                # does not have dependencies? then it's the first task
                for dep in dependencies:
                    aux_dict[dep.__name__] >> aux_dict[task_id]

        return dag

    @classmethod
    def get_airflow_dag(cls, dagname, tasks, default_args={}, catchup=False, concurrency=5, cron=None , collection=None):
        """
        The actual method that has to be called by a DAG file to get the dag.
        params:
            idem as create_dag + add_tasks_to_dag
        returns:
            DAG object
        """
        dag = cls.create_dag(dagname, default_args=default_args, catchup=catchup, concurrency=concurrency, cron=cron)
        dag = cls.add_tasks_to_dag(dag, tasks, collection)
        return dag
```

참고 - [https://towardsdatascience.com/how-to-build-a-dag-factory-on-airflow-9a19ab84084c](https://towardsdatascience.com/how-to-build-a-dag-factory-on-airflow-9a19ab84084c)