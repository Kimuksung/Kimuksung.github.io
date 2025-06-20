---
layout: post
title:  "Kubernetes Airflow Scheduler 개선하기"
author: Kimuksung
categories: [ Airflow ]
#blog post image
image: assets/images/airflow.png
comments: False
featured: True
---

<br>
안녕하세요

오늘은 Kubernetes Executor 환경에서 Airflow 2.9.3을 운영하며 겪은 현상에 대해 공유 드리려고 합니다.

Airflow가 실제 동작하는 영역과 Executor 영역은 Node가 분리되어 있어 서로가 영향을 끼치지 않습니다.
Airflow Web에서 버튼이 상호작용이 안된다던지 하는 현상이 발생되었습니다.
해당 시간에 Scheduler가 꺼졌다가 켜진 이력을 발견하였고 에러는 명확하였습니다. 

원인은 아래와 같습니다.

-> The Node was low on resource: ephemeral-storage 로 인한 Evict
1. Airflow Scheduler에서 모종의 이유로 Node에서 퇴출(Pod의 생명 주기는 한달에 한번 재생성됨)
2. EmptyDir → ephemeral-storage 를 사용중
3. ephemeral-storage Pod의 임시 스토리지가 Node의 임계치보다 더 커졌기 때문에 퇴출 시켰다. ephemeral-storage 의 크기가 왜 커졌을까?
4. /opt/airflow/logs 하위에 dag_processor_manager, scheduler 로그를 적재
5. /opt/airflow/logs 크기가 2GB 넘어가면 Evict

<br>
처음에 생각한 방안은 아래 정도였습니다.
1. Node와 공유되는 공간이 EmptyDir 이외의 다른 Path로 선언
2. 로그 데이터를 7일 주기적으로 삭제 처리한다. (로그가 얼마나 쌓여서 퇴출당한지 모르기 때문)
3. Node Evict 대상이 되지 않도록 한다.

잘 처리가 되었을까요?
- 적용을 했음에도 불구하고 똑같이 발생하였습니다.
- 어려움이 있다면, kubernetes pod에 직접 확인할 수 있다면 좋겠지만 R&R상 접근이 불가합니다.

<br>
다음 방안을 찾아봅니다.

그래도 조금 더 명확해진 이유는 찾았습니다. 

storage가 2GB가 넘으면 퇴출 당한다는 부분과 주기적으로 삭제 되는 path가 scheduler 하위라는점을요.
이제부터는 Docker에다가 직접 테스트를 해봅니다.
4. AIRFLOW__LOGGING__DAG_PROCESSOR_MANAGER_LOG_STDOUT : 로그 적재 -> stdout로 노출
5. AIRFLOW__LOGGING__DAG_PROCESSOR_MANAGER_LOG_LOCATION : scheduler 하위로 위치를 옮겨 삭제 처리
6. logging_config class를 수정하여 삭제 처리 하도록 한다. 
7. logging_level를 info -> warning, error로 처리

여기서 7번까지로 처리하고 싶지 않았습니다. 혹시 모를 이유로 장애가 발생하면 원인을 찾기 어려울 것이라고 판단되어서죠.
다행히도 6번에서 직접 최대 크기를 설정할 수 있고, 백업 갯수까지 지정할 수 있었습니다.

상세 내용은 추가로 작성할 예정입니다.
<br>


