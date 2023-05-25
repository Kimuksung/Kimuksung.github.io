---
layout: post
title:  "Data Pipeline with apache airflow Chatper5 (Task dependency)"
author: Kimuksung
categories: [ Airflow ]
tags: [ Airflow, Branch, Trigger, xcom, taskflow api ]
image: assets/images/airflow.png
# 댓글 기능
comments: False
---


##### Task 의존성이란?
---
Linear chain dependency
- `>>` 연산자를 사용하여 의존성

fan-out/fan-in dependency
- 복잡한 의존성 관계
- `[]` 연산자를 사용하여 의존성 표현
- fan-in = 1 Task → 여러 Upstream Task에 의존
    ```python
    [clean_weather, clean_sales] >> join_datasets
    ```
    
- E.g) (날씨 데이터→날씨 데이터 정제) + (판매 데이터→판매 데이터 정제) → 데이터 세트 구성 → ML 학습 → ML 배포
- start 이후에 fetch_sales,fetch_weather task가 병렬로 실행
    
    ```python
    import airflow
    
    from airflow import DAG
    from airflow.operators.dummy import DummyOperator
    
    start = DummyOperator(task_id="start")
    
    fetch_sales = DummyOperator(task_id="fetch_sales")
    clean_sales = DummyOperator(task_id="clean_sales")
    
    fetch_weather = DummyOperator(task_id="fetch_weather")
    clean_weather = DummyOperator(task_id="clean_weather")
    
    start >> [fetch_sales, fetch_weather]
    fetch_sales >> clean_sales
    fetch_weather >> clean_weather
    ```
    
<br>

##### Branch
---
- 유사한 테스크의 경우에는 조건절을 활용하여 Task 파이프라인 구성 가능
- 하지만, 테스크 자체의 용도가 많이 다른 2가지의 Downstream이 구성된다면?
- 이 과정에서 Task를 명확하게 분리하지 않는다면, Flow를 한번에 알아보는 것은 쉽지 않다.
- **`BranchPythonOperator`** = 작업 결과를 Task id를 반환 → 이를 활용하여 선택 기능 구성

```python
def _pick_erp_system(**context):
    if context["execution_date"] < ERP_CHANGE_DATE:
        return "fetch_sales_old"
    else:
        return "fetch_sales_new"

pick_erp_system = BranchPythonOperator(
  task_id="pick_erp_system", 
	python_callable=_pick_erp_system
)

pick_erp_system >> [fetch_sales_old, fetch_sales_new]
```

- 위와 같이 구성하면 분기타서 실행은 되지만, 지정된 Upstream Task가 전부 완료되지 않았기에 Task는 동작하지 않습니다.

<br>

##### Trigger
---
- 아래 그림과 같이 하나의 Task에서 분기처리 되어 Flow를 실행하다보면, 앞써 배운바를 통해 join_datasets이 동작하지 않는 것을 볼 수 있습니다.
    
    ![https://ifh.cc/g/Vsco3D.png](https://ifh.cc/g/Vsco3D.png)
    
- 이를 해결하기 위해서 Task가 시작하기 위한 조건을 설정할 수 있습니다.
- **`trigger_rule`**
    - all_access = 모든 parent task가 성공해야 해당 Task를 실행
    - none_failed = parent task가 실행 완료 및 실패가 없는 경우 실행
    - all_done = 의존성 Task가 완료되는 즉시 실행
    - one_failed, one_success = 하나의 Upstream Task가 실패,성공하면 모든  Task 를 더 이상 동작하고 싶지 않는 경우(eager rules)
        
        ```python
        join_erp = PythonOperator(task_id="join_erp_branch", trigger_rule="none_failed")
        ```
        

조금 더 명확하게 하기 위해서 Branch가 끝난 뒤에 DummyOperator로 완료된 Task를 적용하여 표현한다.

```python
from airflow.operators.dummy import DummyOperator

join_erp = DummyOperator(task_id="join_erp_branch", trigger_rule="none_failed")
[clean_sales_old, clean_sales_new] >> join_erp_branch
[join_erp_branch, clean_weather] >> join_datasets
```

![https://ifh.cc/g/jXrpZr.png](https://ifh.cc/g/jXrpZr.png)

<br>

##### 조건부 태스크

---

- 특정 조건에 따라 DAG에서 특정 Task를 건너뛸 수 있는 다른 방법
- 위 그림에서 deploy_model Task를 보고 모델이 실제로 배포되었는지 알 수 있을까요? → NO
- **`AirflowSkipException()` =** 해당 Task와 모든 다운스트림 태스크를 Skip 처리

```python
def _latest_only(**context):
    now = pendulum.now("UTC")
    left_window = context["dag"].following_schedule(context["execution_date"])
    right_window = context["dag"].following_schedule(left_window)

    if not left_window < now <= right_window:
        raise AirflowSkipException()

latest_only >> deploy_model
```

![https://ifh.cc/g/rpsXkV.png](https://ifh.cc/g/rpsXkV.png)

가장 최근에 실행한 DAG만 실행 할 수 있도록 내부 Operator를 지원하여 줍니다.

**`LatestOnlyOperator`** = 조건부 배포를 하기 위해 만들어진 기능

```python
from airflow.operators.latest_only import LatestOnlyOperator

latest_only = LatestOnlyOperator(task_id="latest_only", dag=dag)
latest_only >> deploy_model
```

Task 실행 과정

- Airflow → DAG 실행 →  지속적으로 Task를 확인 → 실행 가능하다고 판단되면 즉시 스케줄러에 의해 선택 및 예약 → Slot이 남아있으면 즉시 실행

Airflow Task 실행 시기를 어떻게 결정하는가?

- 의존적인 Task가 모두 성공적으로 처리되어야 한다.
- Trigger = **`all_access`**인 경우에는 의존성이 모두 해결 → 실행 가능 준비 → 다음 DAG 의존성 삭제 → 전체 DAG Task 실행 될 때까지 실행
    - 위 처럼 **`Propagation`** = Upstream Task가 DownStream Task에 영향을 줍니다.
    - 의존성 Task들이 실패,스킵됨으로써 의존성이 있는 Task에 영향을 끼친다.
- Trigger = **`None_failed`** 인 경우 Upstream Task 완료 여부만 확인
- 실패는 Propagation이지만, 스킵은 Propagation X

<br>

##### XCOM
---
- Task간 데이터 공유 ( 작은 데이터 )
- 일반적으로 Message(state)를 교환
- xcom_push
- xcom_pull
- 일부 오퍼레이터는 Xcom 값을 자동으로 게시 ( PythonOperator-Return )
    
    ```python
    def _train_model(**context):
        model_id = str(uuid.uuid4())
        return model_id
    ```
    
- 

```python
def _train_model(**context):
    model_id = str(uuid.uuid4())
    context["task_instance"].xcom_push(key="model_id", value=model_id)

def _deploy_model(**context):
    model_id = context["task_instance"].xcom_pull(
        task_ids="train_model", key="model_id"
    )
    print(f"Deploying model {model_id}")

train_model >> deploy_model
```

단점

- Xcom을 활용하여 값을 사용하게 된다면, Task간 의존성이 나타나지 않아 묵시정 의존성이 나타난다.
- Operator 원자성을 무너뜨리는 패턴이 발생 할 수 있다.
- E.g) API Token 발급 후 Xcom을 통하여 전달 - 이 경우 시간이 만료된다면 원자성이 깨진다.
- 모든 값을 Serialize 하여 처리 → Lambda, Multi process 관련 Class는 불가능
- Airflow metastore에 저장되며, 사용되는 DB에 따라 크기 제한
    - SQLite - 2GB / Postgresql - 1GB / Mysql - 64KB

- Airflow 2부터는 커스텀 백엔드에 저장 할 수 있도록 추가
- BaseXcom class가 상속 처리 + 값을 Serialize,deserialzie 처리 하기 위한 Method를 구현

```
from typing import Any
from airflow.models.xcom import BaseXCom
from airflow.providers.amazon.aws.hooks.s3 import S3Hook

class S3XComBackend(BaseXCom):
	 PREFIX = "xcom_s3"
	 BUCKET_NAME = os.environ.get("S3_XCOM_BUCKET_NAME")
		@staticmethod
		def serialize_value(value: Any):
		    if isinstance(value, pd.DataFrame):
		        hook = S3Hook()
		        key = f"{str(uuid.uuid4())}.pickle"
		        filename = f"{key}.pickle"
		        value.to_pickle(filename, index=False)
		        hook.load_file(
		            filename=filename,
		            key=key,
		            bucket_name=S3XComBackend.BUCKET_NAME,
		            replace=True
		        )
		        value = f"{S3XComBackend.PREFIX}://{S3XComBackend.BUCKET_NAME}/{key}"
		
		    return BaseXCom.serialize_value(value)

		@staticmethod
		def deserialize_value(result) -> Any:
		    result = BaseXCom.deserialize_value(result)
		
		    if isinstance(result, str) and result.startswith(S3XComBackend.PREFIX):
		        hook = S3Hook()
		        key = result.replace(f"{S3XComBackend.PREFIX}://{S3XComBackend.BUCKET_NAME}/", "")
		        filename = hook.download_file(
		            key=key,
		            bucket_name=S3XComBackend.BUCKET_NAME,
		            local_path="/tmp"
		        )
		        result = pd.read_csv(filename)
		
		    return result
```

<br>

##### Taskflow API

---

- Task 및 의존성을 정의하기 위해서 decorator 기반 Api 지원
- `@task` decorator를 활용하여 변환
- 앞에서 본것과 같이 Airflow의 Xcom을 전달하기 위해 Task를 일일히 설정 및 구현해줘야 한다.
- 간단하면서도 확실하게 표현하는 방법
- 하지만 항상 사용하는 것은 아니다
    - PythonOperator를 사용하여 구현되는 Python Task로 제한
    - 다른 Operator와 연결할 때에는 Task 및 Dependency 정의해야 한다.
    - 혼용하며게 되면 직관적이지 않기 때문에 **`PythonOperator만 모여있는 경우 사용하는게 좋다.`**

```python
import uuid
import airflow
from airflow import DAG
from airflow.decorators import task

	@task
  def train_model():
      model_id = str(uuid.uuid4())
      return model_id

  @task
  def deploy_model(model_id: str):
      print(f"Deploying model {model_id}")

  model_id = train_model()
  deploy_model(model_id)
```