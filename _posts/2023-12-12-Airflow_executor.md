---
layout: post
title:  "Airflow 동작 원리(Executor)"
author: Kimuksung
categories: [ Airflow ]
tags: [ Airflow ]
image: assets/images/airflow.png
comments: False
---

##### Componenets
- Webserver : Flask Server
- Scheduler : Daemon
- Metastore : database where metadata are stored
- Executor : class defining **`How`** your tasks should be executed
- Worker : process/subprocess **`executing`** task

<img src="https://kimuksung.github.io/assets/images/Airflow_동작원리1.png"/>
<br/>

<img src="https://kimuksung.github.io/assets/images/Airflow_동작원리2.png"/>
<br/>

##### Executors
- Excutor에 따라 Task의 동작원리가 바뀐다.
- 일반적으로 [Task1,Task2] → Task3로 가는 구조의 DAG가 있다고 하여보자.

<img src="https://kimuksung.github.io/assets/images/Airflow_동작원리3.png"/>
<br/>

- One Node Architecture
    - Executor와 Queue는 붙어있다.
    - Sequential Executors
    - LocalExecutor
        
    <img src="https://kimuksung.github.io/assets/images/Airflow_동작원리9.png"/>
<br/>
        
- Multi Node Architecture
    - CeleryExecutor
    - K8sExecutor
        
    <img src="https://kimuksung.github.io/assets/images/Airflow_동작원리9.png"/>
<br/>     
        

- Sequential Executors
    - default executor
    - 스케줄러 내부에서 동작
    - 한번에 하나의 테스크 인스턴스
    - Sqlite 를 사용
    - 장점
        - 별도의 설치가 필요 없다.
        - 가볍고 싸다.
    - 단점
        - 확장성 X
        - 한번에 하나의 Task만 실행 시키기에 느리다.
        - SPOF 문제
        - Production에 적합하지 않다.
    
    <img src="https://kimuksung.github.io/assets/images/Airflow_동작원리4.png"/>
<br/>
    
- Local Executors
    - 스케줄러 내부에서 동작
    - 여러 테스크 인스턴스 동시에 실행 가능
    - 장점
        - 설치가 쉽다.
        - 가볍고 싸다.
        - 여러 테스크를 동시에 동작 가능
    - 단점
        - 확장성이 없다.
        - Production 환경에 부적합
        - SPOF 문제
        
    <img src="https://kimuksung.github.io/assets/images/Airflow_동작원리5.png"/>
<br/>
        

- Celery Executor
    - task를 동작시키는 machine이 별도로 존재
    - Task Queue를 분산처리
    - 장점
        - 수평적 확장이 가능
        - Falut Tolerant
        - Production에 적합
    - 단점
        - 설치 시 시간이 소요
        - 테스크가 없다면, resource가 낭비
        - 비용 많이 든다.

    <img src="https://kimuksung.github.io/assets/images/Airflow_동작원리6.png"/>
<br/>

- Kunbernetes Executor
    - Task를 동작시키는 Pod가 별도로 존재
    - K8s API를 사용하여 pod를 관리
    - 장점
        - 0으로 확장 가능
        - Falut Tolerant
        - Task별로 resource를 할당 가능
        - 비용/리소스가 효율적
    - 단점
        - Task 별로 Pod 시작시키는 시간(Overhead)이 든다.
        - K8S 지식이 필요

<img src="https://kimuksung.github.io/assets/images/Airflow_동작원리7.png"/>
<br/>

Architecture

- One Node
    - Executor와 Queue는 붙어있다.
- Multi Node(Celery)

##### 참조
---
https://www.youtube.com/watch?v=TQIInLmKM4k