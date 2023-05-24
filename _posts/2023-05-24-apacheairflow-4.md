---
layout: post
title:  "Data Pipeline with apache airflow Chatper4 (Template, Operator)"
author: Kimuksung
categories: [ Airflow ]
tags: [ Airflow, Cron, execution_date, partitioning, backfill, 원자성, 멱등성 ]
image: assets/images/airflow.png
# 댓글 기능
comments: False
---

안녕하세요

오늘은 **Task Operator Keyword Template화 하기 위한 과정**을 알아보겠습니다.  
Airflow 2.x 이후부터는 Operator는 별도의 pip package를 통해 설치된다는 점  
airflow package안에는 bashoperator, PythonOperator만 존재.  

##### **Template**
---
- Template 작업은 런타임에 실행
- Python Operator는 다른 Operator들과 다르게 동작 ( 변수 호출 가능하도록 )
- 템플릿 인수 결과는 render에서 확인 가능

- File Template 구성 시
- 기본적으로 DAG 파일 경로만 검색
- Jinja를 통해서 다른 파일 경로를 검색하고 싶다면, **`template_searchpath`** 인수 설정하면 된다.  
```python
dag = DAG(
    dag_id="listing_4_20",
    start_date=airflow.utils.dates.days_ago(1),
    schedule_interval="@hourly",
    template_searchpath="/tmp",
    max_active_runs=1,
)
```

##### BashOperator
---
- Jinja Template을 동적으로 넣는 방법 ⇒ 이중 중괄호 런타잉 시 삽입될 변수 
  ```python
  get_data = BashOperator(
      task_id="get_data",
      bash_command=(
          'curl -o /tmp/wikipageviews.gz '
          'https://dumps.wikimedia.org/other/pageviews/'
          '{ { execution_date.year }}/'
          '{ { execution_date.year }}-{ { '{:02}'.format(execution_date.month) }}/'
          'pageviews-{ { execution_date.year }}'
          '{ { '{:02}'.format(execution_date.month) }}'
          '{ { '{:02}'.format(execution_date.day) }}-'
          '{ { '{:02}'.format(execution_date.hour) }}0000.gz'
      ),
      dag=dag,
  )
  ```
- Airflow Datetime ⇒ Pendulum 라이브러리 사용
  ```python
  from datetime import datetime
  import pendulumn
  
  datetime.now().year
  > 2023
  pendulumn.now().year
  > 2023
  ```
    
- 특정 시간 대 앞자리를 0으로 채우기 위해서는 패딩 문자열 사용
  ```python
  { {'{:02}'.format(execution_date.hour) }}
  ```
    

##### Python Operator
---
- Bash Operator와 유사하나, python_callable 인수에 callable Object를 제공
- Airflow Version에 따라 함수 내에 `**kwagrs` 전달하기 위한 설정 값이 다르다.
    - Airflow 1.x ⇒ Provide_context = True
    - Airflow 2.x ⇒ 별도 Parameter를 설정할 필요 없다.
- Function은 Keyword를 받아 사용(**kwargs)

```python
# keyword 인수
def _print(**kwargs):
	print(kwargs)

# text_context 저장 의도를 표현하기 위함
def _print_context(**context):
	start = context["execution_date"]
	end = context["next_execution_date"]
	print(f"{start} / {end}")

print_context = PythonOperator(
	task_id = "print_context",
	python_callable = _print_context,
	dag = dag
)
```

```python
def _get_data(execution_date, **context):
    year, month, day, hour, *_ = execution_date.timetuple()
    url = (
        "https://dumps.wikimedia.org/other/pageviews/"
        f"{year}/{year}-{month:0>2}/pageviews-{year}{month:0>2}{day:0>2}-{hour:0>2}0000.gz"
    )
    output_path = "/tmp/wikipageviews.gz"
    request.urlretrieve(url, output_path)

get_data = PythonOperator(task_id="get_data", python_callable=_get_data, dag=dag)
```

Operator에 변수 제공
- `op_args`, `op_kwargs` 사용
- 아래는 op_args를 통해 List를 전달하는 예시
    ```python
    get_data = PythonOperator(
    	task_id="get_data", 
    	python_callable=_get_data, 
    	op_args = ["test/test.py"]
    	dag=dag
    )
    
    python
    get_data = PythonOperator(
    	task_id="get_data", 
    	python_callable=_get_data, 
    	op_kargs = {"output_path":"test/test.py" }
    	dag=dag
    )
    
    ```
    

Render  
- Template Keyworkd 오류 디버깅 시 사용
- Task → Render에 들어가서 확인 가능하다.
    
    ![https://ifh.cc/g/74gTYp.png](https://ifh.cc/g/74gTYp.png)
    
- CLI 사용하여 Template Rendering은 아래와 같다.
    ```python
    $ airflow tasks render [dag_id] [task_id] [desired_execution_date]
    ```
    

##### 다른 시스템과 연결하기
---
- Airflow Task ⇒ 설정에 따라 물리적으로 서로 다른 컴퓨터에서 독립적으로 실행 → 메모리에서 데이터 공유 불가
- Xcom이라는 매커니즘을 제공 → Metastore에서 선택 가능한 pickable(직렬화 프로토콜) Object 저장 후 나중에 읽을 수 있다.
- 크기가 작은 경우 Xcom Pickling 처리가 알맞다.
- 크기가 크게 되면 Airflow 외부에 데이터를 저장 ( 필요 시 페이징 )
- Pickle이 불가능한 Object = DB Connection, File Handler

Postgres 연결하기

- Airflow 입장에서는 Postgres는 외부 시스템
- 대부분의 오케스트레이션의 시스템고 같이, 여러 오퍼레이터를 통해 광범위한 연결 지원
- **`Hook`** = 연결 생성, 쿼리 전송, 연결 종료를 처리하는 작업 `인스턴스`
- Operator = 무엇을 해야할지
- Hook = Operator가 정한 무엇을 어떻게 해야 할지

```python
$ pip install apache-airflow-providers-postgres
```

```python
from airflow.providers.postgres.operators.postgres import PostgresOperator

dag = DAG(
    dag_id="listing_4_20",
    start_date=airflow.utils.dates.days_ago(1),
    schedule_interval="@hourly",
    template_searchpath="/tmp",
    max_active_runs=1,
)

write_to_postgres = PostgresOperator(
    task_id="write_to_postgres",
    postgres_conn_id="my_postgres",
    sql="postgres_query.sql",
    dag=dag,
)
```

![https://ifh.cc/g/82znvh.png](https://ifh.cc/g/82znvh.png)