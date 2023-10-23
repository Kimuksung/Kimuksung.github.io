---
layout: post
title:  "Airflow Scheduling"
author: Kimuksung
categories: [ Airflow ]
tags: [ Airflow, start_date, execution_date ]
image: assets/images/airflow.png
# 댓글 기능
comments: False
---

##### Airflow
---
N 배치인 경우 N전 기준으로 돈다고 이해하면 된다.

스케줄링 시간을 빠르게 파악하고 싶다면 → https://crontab-generator.com/ko

```python
* * * * *
```

- **각각의 별표(``)는 다음과 같은 의미를 가집니다.**
    - 분(Minute) : `0~59`
    - 시간(Hour) : `0~23`
    - 일(Day) : `1~31`
    - 월(Month) : `1~12`
    - 요일(Day of the week) : `0~7` (0과 7은 일요일)

하루에 한 번 도는 일배치라고 생각해보자.

```python
0 1 * * * 
```

**2023-10-23 1시에는 2023-10-22 1시 기준의 Job이 실행된다.**

- [start_date 상세 링크](https://kimuksung.github.io/airflow-startdate/)
- [execution_date 상세 링크](https://kimuksung.github.io/airflow_exectiondate/)