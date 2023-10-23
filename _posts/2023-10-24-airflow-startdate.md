---
layout: post
title:  "Airflow Start_date"
author: Kimuksung
categories: [ Airflow ]
tags: [ Airflow, start_date ]
image: assets/images/airflow.png
# 댓글 기능
comments: False
hidden: True
---

<br>

##### Start_date
---
<aside>
💡 DAG가 시작되는 고정 시간
</aside>

- start_date에 실행된다는 의미가 아니다.
- 현재 시간이 start_date보다 이전이면 DAG는 시작하지 않는다.
- DAG의 Start_date와 Airflow UI에서의 Start_date는 의미가 다르다.
- Airflow UI Start_date ⇒ Task가 실행되는 날짜
- start_date = 2023-10-23 00시라면 해당 날짜 이후부터 시작되는 DAG를 만들고 실제 동작은 이 시간이 지난 2023-10-23 1시에 시작된다는 것이다.

```python

dag = DAG(
	dag_id = 'scheduling',
	start_date = datetime(2023, 10, 23),
	schedule_interval = '0 1 * * *'
    )
```