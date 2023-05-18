---
layout: post
title:  "Data Pipeline with apache airflow Chatper1"
author: Kimuksung
categories: [ Airflow ]
tags: [ Airflow ]
image: assets/images/airflow.png
# 댓글 기능
comments: False
---

안녕하세요

오늘은 데이터 파이프라인을 구축하기 위해 Airflow를 어떻게 사용하는지에 대해 알아보려고 합니다.   

<br>

##### 데이터 파이프라인
---
- DAG에 Task와 그에 대한 의존성 정의 + 의존성을 가진 Task를 병렬 처리

##### DAG
---
- Python Script로 구성 = 파이프라인 구성

<br>

##### Airflow 3가지 구성 요소

###### Airflow 스케줄러
---
- Dag 분석, 스케줄이 지난 경우 Worker에 Task 에약
1. Dag를 작성 → Scheduler가 Dag file 분석 → Task,Dependency, 예약 주기를 확인
2. 마지막 Dag까지 확인 후, 예약 주기가 현재 시간 이전이라면 실행 예약
3. 예약된 Task들에 대해서 Dependency(Upstream)을 check → 의존성이 검증 → Q에 적재
4. 1번 단계로 되돌아간 뒤, 잠시 대기

<br>

##### Airflow Worker

---

- 예약된 Task를 선택하고 실행
- Worker의 pool은 Q에 있는 Task를 선택하고 실행.
- 병렬로 실행, 실행 결과 지속적으로 추적
- 과정의 모든 결과를 Airflow MetaStroe(진행 상황 및 로그)로 전달

<br>

##### Airflow Web server

---

- DAG 시각화, 실행, 결과 확인 할 수 있는 UI

![https://ifh.cc/g/1jGSZr.jpg](https://ifh.cc/g/1jGSZr.jpg)

![https://ifh.cc/g/RAxQss.jpg](https://ifh.cc/g/RAxQss.jpg)

<br>

##### 언제 Airflow를 사용해야 하는가?

---

- 배치 지향 데이터 파이프라인
    - Python code로 구현 가능한 파이프라인은 전부 구현 가능
    - 확장성이 좋으며, DB, Cloud와 손쉽게 연동 가능
    - 정기적 실행,증분 처리가 필요한 경우
    - 백필 기능 → 과거 데이터 쉽게 재처리
    - UI를 지원하여, 실행 결과를 모니터링 / 오류 디버깅이 쉽다.
- 오픈 소스이다.

이럴 때는 사용 X

- Streaming Workflow, Pipeline 처리
- 추가,삭제가 빈번한 동적 파이프라인 ( 실행되는 동안 구조가 변경되지 않는 파이프라인에 적합 )
- Python 프로그래밍 경험이 부족한 경우
- Pipeline 크기가 커지는 경우 → 복잡해진다.
