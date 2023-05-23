---
layout: post
title:  "Data Pipeline with apache airflow Chatper2 (DAG, Operator, Task)"
author: Kimuksung
categories: [ Airflow ]
tags: [ Airflow, DAG, Operator, Task ]
image: assets/images/airflow.png
# 댓글 기능
comments: False
---

안녕하세요  
오늘은 Airflow Dag 구조에 대해 설명하려고 합니다.  

##### DAG

---

- dag_id
- start_date

```bash
dag = DAG( 
    dag_id = "DAG_NAME", 
    start_date = airflow.utils.dates.days_ago(n),
		schedule_interval = None,
)
```

##### Operator

---

- 각 오퍼레이터는 하나의 Task 구성
- Operator는 독립적으로 실행
- 의존성(Dependency) - 순서를 정의하여서 사용 `>>`

```bash
cmd_ls3 = BashOperator(
	task_id='request_redshift_connect',
	bash_command='nc -zv wclub-data.cie4a1zjuetm.ap-northeast-2.redshift.amazonaws.com 5439',
	dag=dag
)
```

##### Task vs Operator

---

일반적으로 Operator와 Task라는 용어를 동일하게 사용하지만, 사용자 관점에서만 같은 의미

Task 

- 작업의 올바른 실행을 보장하기 위한 Operator 매니저
- Operator 상태를 관리
- 사용자에게 상태 변경(시작/완료) 알려주는 Airflow 내장 Component

Operator

- 사용자는 DAG와 Operator를 이용
- Operator는 단일 작업 수행 역할
- DAG는 Operator set에 대한 실행을 Orchestration(조정,조율)하는 역할
- Operator 시작과 정지, 완료 후 다음 Task 시작

![https://ifh.cc/g/56nbyF.png](https://ifh.cc/g/56nbyF.png)

##### Airflow 실행

---

- Python
    
    ```bash
    $ pip install apache-airflow
    
    $ airflow db init
    $ airflow users create --username "username" --password "pw" --firstname "name"
    --lastname "Admin" --role Admin --email "mail@co.kr"
    
    $ airflow webserver
    $ airflow scheduler
    $ cd ~/airflow/dags
    ```