---
layout: post
title:  "Airflow Catchup"
author: Kimuksung
categories: [ Airflow ]
tags: [ Airflow, start_date ]
image: assets/images/airflow.png
# 댓글 기능
comments: False
---

##### Catchup
---
- True : Start_date부터 현재시점까지의 모든 스케줄링을 모두 실행
- False : Start_date가 아닌 현재 기준으로부터 모든 스케줄링 실행
- 이 때, 모든 Task가 동시에 실행될 수 있으니, 주의 필요

```python
with DAG(
    dag_id="example_dag",
    start_date=datetime(2023, 11, 7), 
    max_active_runs=1,
    default_args={
        "retries": 1,
        "retry_delay": timedelta(minutes=3),
    },
    catchup=False
) as dag:
```

<a href="https://ibb.co/hgYCxfY"><img src="https://i.ibb.co/qDJxL5J/2023-11-07-6-20-41.png" alt="2023-11-07-6-20-41" border="0"></a>

한번에 실행될 때 문제를 방지하기 위해 알아야 할 값들

- max_active_runs : DAG 수준에서 설정/ Catchup 중 실행될 수 있는 active run
- depends_on_past : 최근 DAG에 의존되어 마무리가 되어야 실행이 된다.
- wait_for_downstream : 다음 DAG를 실행하려면 전체 Task가 실행되어야 한다.
- catchup_by_default : config 파일에서 설정, DAG 기본값 설정 ( 일반적으로 False )