---
layout: post
title:  "Airflow BashOperator"
author: Kimuksung
categories: [ Airflow ]
tags: [ Airflow, BashOperator ]
image: assets/images/airflow.png
# 댓글 기능
comments: False
hidden: True
---

##### Bash Operator
---
<aside>
💡 Bash shell script를 실행하는 Operator
</aside>

- 파이썬 코드가 아닌 스크립트를 정기적으로 실행이 가능하다.

```python
classairflow.operators.bash.BashOperator(
    *, 
    bash_command: str, 
    env: Optional[Dict[str, str]] = None, 
    output_encoding: str = 'utf-8', 
    skip_exit_code: int = 99, 
    cwd: str = None, **kwargs
)
```

- output_encoding : bash command 인코딩 방식
- skip_exit_code : `None` - 0이 아닌 모든 값은 실패 / `99` - task exit 한 경우에 skip 처리
- cwd : command가 실행된 위치 ( cd 와 같은 번거로움을 줄여줌 )

```python
from airflow.operators.bash import BashOperator
```

```python
from airflow import DAG
from airflow.operators.bash_operator import BashOperator
om airflow.operators.dummy import DummyOperator
from datetime import datetime

dag = DAG(
    'dag name',
    description='testing for bashoperator',
    schedule_interval=None,
    start_date=datetime(2023, 10, 23),
    catchup=False
)

start_task = DummyOperator(task_id='start_task', dag=dag)

task1= BashOperator(
    task_id='task1',
    bash_command='python -c "print(\'Hello, world!\')"',
    output_encoding= 'utf-8'
    dag=dag
)

end_task = DummyOperator(task_id='end_task', dag=dag)

start_task >> task1 >> end_task 
```

참조
---
https://medium.com/@agusmahari/airflow-getting-started-with-the-bashoperator-in-airflow-a-beginners-guide-to-executing-bash-1f80d25c4602