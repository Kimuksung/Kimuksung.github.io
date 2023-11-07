---
layout: post
title:  "Airflow Start_date"
author: Kimuksung
categories: [ Airflow ]
tags: [ Airflow, start_date ]
image: assets/images/airflow.png
# 댓글 기능
comments: False
---

<br>

##### Start_date
---
<aside>
💡 DAG가 시작되는 고정 시간
</aside>

- start_date : DAG가 데이터를 읽을 시간
- scheduler가 지금 실행해야한다고 인지해야하는 시간
- start_date에 실행된다는 의미가 아니다.
- 현재 시간이 start_date보다 이전이면 DAG는 시작하지 않는다.
- DAG의 Start_date와 Airflow UI에서의 Start_date는 의미가 다르다.
- Airflow UI Start_date ⇒ Task가 실행되는 날짜
- start_date = 2023-10-23 00시라면 해당 날짜 이후부터 시작되는 DAG를 만들고 실제 동작은 이 시간이 지난 2023-10-23 1시에 시작된다는 것이다.
- 아래 실제 Airflow 그림에서는 **Started = Start_date**을 의미한다.

```python

dag = DAG(
	dag_id = 'scheduling',
	start_date = datetime(2023, 10, 23),
	schedule_interval = '0 1 * * *'
    )
```

<a href="https://ibb.co/2MWRVkC"><img src="https://i.ibb.co/Jpm86sG/2023-11-07-6-19-51.png" alt="2023-11-07-6-19-51" border="0"></a><br /><a target='_blank' href='https://nonprofitlight.com/ny/new-york/garden-of-eden-foundation-inc'></a><br />

##### Execution_date
---
- Execution_date : 실제 DAG가 실행되도록 설계된 시간 ( 실제 실행되는 시간 X )
- Logical date라고 불리며, 코드가 동작하도록 맞춘 시간
- Airflow 그림에서는 **RUN = Execution_date**를 의미한다.


<a href="https://ibb.co/xfzXhdM"><img src="https://i.ibb.co/s6sbjzF/2023-11-07-6-03-22.png" alt="2023-11-07-6-03-22" border="0" width = 500></a>


참조
---
- https://medium.com/nerd-for-tech/airflow-catchup-backfill-demystified-355def1b6f92