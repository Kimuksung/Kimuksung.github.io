---
layout: post
title:  "Airflow execution_date"
author: Kimuksung
categories: [ Airflow ]
tags: [ Airflow, execution_date ]
image: assets/images/airflow.png
# 댓글 기능
comments: False
hidden: True
---

<br>

##### execution_date
---
- 실제 실행 날짜가 아니다.
- 스케줄러의 관점에서 일배치라면 `Schedule Interval`(어제 날짜부터 ~ 실제 DAG가 실행되는 실행 날짜)가 필요하다. 이 때 어제 날짜가 `execution_date`가 되는 것이다.
- execution_date는 처리해야할 **실행 날짜**를 의미한다.
- 멱등성을 위하여 존재 해당 기점으로 돌려주는 역할
- 2.2 버전 이후부터는 execution_date를 deprecated
    - logical_date = execution_date
    - data_interval_start = execution_date
    - next_execution_date = next_logical_date, data_interval_end
    - prev_execution_date = prev_logical_daste
    

![](https://blog.bsk.im/content/images/2021/03/new_variables.png)

##### 예시
---
```python
0 3 * * *
start_date = 2023-10-23

# 첫 task 실행 시작 = 2023-10-23 03:00:00
# 첫 execution_date = 2023-10-22 03:00:00
```

##### 참조
---
- https://blog.bsk.im/2021/03/21/apache-airflow-aip-39/
- https://cwiki.apache.org/confluence/display/AIRFLOW/AIP-39+Richer+scheduler_interval