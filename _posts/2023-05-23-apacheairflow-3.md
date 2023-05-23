---
layout: post
title:  "Data Pipeline with apache airflow Chatper3 (Scheduling, Template, Backfill, 원자성, 멱등성)"
author: Kimuksung
categories: [ Airflow ]
tags: [ Airflow, Cron, execution_date, partitioning, backfill, 원자성, 멱등성 ]
image: assets/images/airflow.png
# 댓글 기능
comments: False
---

오늘은 Airflow 기본 필수 개념인 스케줄링과 멱등성,원자성에 대해 알아보려고 합니다.

<br>

#### 스케줄링
---
- 스케줄 간격을 설정하여 DAG 실행
- 하나의 스케줄 간격 작업은 해당 주기의 작업이 끝나면 시작

interval-based(간격 기반)
- 매번 반복되는 작업의 시작과 끝이 정해져 있는 경우

point-based(시점 기반)
- 작업이 시작되는 시점은 알지만, 언제를 끝으로 해야하는지 정하지 않은 경우

![https://ifh.cc/g/hCaoW0.jpg](https://ifh.cc/g/hCaoW0.jpg)

![https://ifh.cc/g/D7NTNb.png](https://ifh.cc/g/D7NTNb.png)

<br>

##### Cron
---
- 분 시간 일 월 요일
- 대부분의 상황에서 주기적으로 실행을 도와준다.
- Cron 계산하여 나타내주는 URL = [https://crontab.guru/](https://crontab.guru/)

- 특정 빈도 기반은 스케줄링하기에 Cron이 적합하지 않다.
- timedelta를 이용
    
    ```bash
    dag = DAG( 
        dag_id = "DAG_NAME", 
        start_date = airflow.utils.dates.days_ago(n),
    		schedule_interval = dt.timedelta(days=3),
    )
    ```
    

##### 데이터 증분 처리하기
---
- 데이터가 점차적으로 증가하는데, 처음부터 데이터를 다시 불러오면 효율적이지 못하다.

```bash
event = BashOperator(
	task_id='event',
	bash_command=(
		'mkdir -p /data &&',
		"curl -o /data/event.json "
		"url "
		"start_date=2023-05-13&"
		"end_date=2023-05-14"
)
	dag=dag
)
```

동적 시간 참조
- execution_date = 스케줄 간격으로 실행되는 시작 시간
- previous_execution_date = 과거의 스케줄 시작 시간
- next_execution_date = 다음 스케줄 시작 시간
- ds = execution_date → YYYY-MM-DD
- ds_nodash = execution_date → YYYYMMDD
- next_ds = next_execution_date → YYYY-MM-DD
- next_ds_nodash = next_execution_date → YYYYMMDD

```bash
event = BashOperator(
	task_id='event',
	bash_command=(
		'mkdir -p /data &&',
		"curl -o /data/event.json "
		"url "
		"start_date={{execution_date.strftime('%Y-%m-%d')}}"
		"end_date={{next_execution_date.strftime('%Y-%m-%d')}}"
)
	dag=dag
)
```
```bash
event = BashOperator(
	task_id='event',
	bash_command=(
		'mkdir -p /data &&',
		"curl -o /data/event.json "
		"url "
		"start_date={{ds}}"
		"end_date={{next_ds}}"
)
	dag=dag
)
```

<br>

##### 파티셔닝
---
데이터 세트를 더 작고 관리하기 쉬운 조각으로 나누는 작업

`파티션` = 데이터 세트의 작은 부분

보일러플레이트 코드(boilerplate) = 입력 및 출력에 대한 값을 템플릿화하여 사용

```bash
python_operator = PythonOperator(
    task_id='test',
    python_callable=myfunc,
    params = {
			"input_path":"/data/event/{{ds}}.json",
			"output_path":"/data/event/{{ds}}.csv"
		},
		# Airflow version 1.x 이라면 아래 추가
		# provide_text = True,
    dag=dag,
)
```

<br>

##### 백필
---
- 과거 데이터 간격을 메꾸기 위함
- 과거 데이터 세트 로드,분석하기 위함
- 주의점
    - API가 최근 최대 30일까지의 데이터만 제공 → 이전 데이터를 백필 처리하여도 소용이 없다.
    - Resource에 많은 부하를 유발

과거 시점의 작업 실행

- Airflow는 기본적으로 아직 실행되지 않은 과거 스케줄 간격을 예약하고 실행
- E.g) 7일전에 하루 주기로 시작하는 DAG를 만들었다면, DAG가 활성화 대는 순간 순차적으로 7,6,5,4,3,2,1이 모두 실행
- `catchup` = Boolean type → 가장 최근 스케줄 간격에 대해서만 실행
    - default 값은 configuration file에서 catchup_by_default 값을 설정하여 제어
    
    ```bash
    dag = DAG( 
      dag_id = "DAG_NAME", 
      start_date = airflow.utils.dates.days_ago(n),
    	schedule_interval = dt.timedelta(days=3),
    	catchup = False
    )
    ```
    
    ![https://ifh.cc/g/tOKPBA.png](https://ifh.cc/g/tOKPBA.png)
    

<br>

#### Airflow Design

##### 원자성(atomicity)
---
- 데이터 베이스 원자성 트랜잭션과 동일
- 모두 발생하거나 전혀 발생하지 않는 데이터베이스 작업과 동일
- 강한 의존성을 가진 경우(E.g API 인증 토큰 + API Call ) 같은 경우는 하나의 태스크를 하는것이 유리
- 하나의 Task에서 2개의 작업은 원자성을 무너뜨린다.

```python
# 원자성 무시
# 계산하는 로직 + 이메일 전송 로직
def _calculate_stats(**context):
    """Calculates event statistics."""
    input_path = context["templates_dict"]["input_path"]
    output_path = context["templates_dict"]["output_path"]

    events = pd.read_json(input_path)
    stats = events.groupby(["date", "user"]).size().reset_index()

    Path(output_path).parent.mkdir(exist_ok=True)
    stats.to_csv(output_path, index=False)

    _email_stats(stats, email="user@example.com")

fetch_events >> calculate_stats
```

```python
# 원자성
def _send_stats(email, **context):
    stats = pd.read_csv(context["templates_dict"]["stats_path"])
    email_stats(stats, email=email)

send_stats = PythonOperator(
    task_id="send_stats",
    python_callable=_send_stats,
    op_kwargs={"email": "user@example.com"},
    templates_dict={"stats_path": "/data/stats/{{ds}}.csv"},
    dag=dag,
)

fetch_events >> calculate_stats >> send_stats
```

##### **멱등성(idempotency)**
---
- 동일한 입력 → 동일한 태스크 여러 번 호출해도 결과에 영향 X
- 일관성과 장애 처리 보장 ( 작업을 재실행 할 수 있도록 보장 )
    
    ![https://ifh.cc/g/wTaAdz.png](https://ifh.cc/g/wTaAdz.png)