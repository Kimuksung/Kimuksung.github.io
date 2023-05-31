---
layout: post
title:  "Distributed System - Global Lock"
author: Kimuksung
categories: [ Distributed System ]
tags: [ Global Lock ]
image: "https://ifh.cc/g/sCSX4x.png"
comments: False
---


분산처리 시스템에서의 동시성과 단일 노드에서 동시성을 제어하기 위한 Lock들을 소개하고 어떤 점이 다른지 소개 드리려고 합니다.
##### Mutex
---
- 특정 쓰레드, 프로세스가 공유 Resource에 접근하는 것을 동기화 하는데 사용.
- 하나의 쓰레드가 Mutex Lock하게 되면, 다른 쓰레드들은 Release 할 때까지 Wait해야 한다.
- 공유데이터에 대해 동시 접근을 제어하는데 사용되며, OS 커널에서 지원

<br>

##### execlusive lock
---
- Database, File System에서 하나의 Trnasaction은 특정 항목을 읽고 쓰기를 보장합니다.
- 격리성 정도에 따라, 모든 읽기나 쓰기 작업은 락이 해제 될 떄까지 대기

<br>

##### Global lock
---
- 여러 쓰레드,프로세스가 동시에 데이터에 엑세스하거나 수정하지 못하게 하는 동기화 메커니즘
- Resource에 대해 동시 엑세스를 방지 및 데이터 일관성 보장
- Mutex와 같은 개념이지만, Mutex는 하나의 OS내에서 동작하는 경우에 해당
- 일반적으로 서버를 다중화하여 구성하기 때문에 부하 분산 처리 시 이를 사용
- 분산 처리 환경과 같이 여러 노드들이 동시에 수정하는 경우를 막기 위해 Lock 정보를 어딘가 공통적으로 보관
- 어딘가? = Mysql-Namenode, Redis, Zookeeper ..

E.g) 티켓 선착순 시스템이라고 하여 보자.
Mutex
총 티켓은 30개인데, 이벤트가 발생하자마자 50명의 유저가 요청을 하였다. 이 때에 하나의 OS인 경우에서는 Thread에 Lock을 잡고 있어 다른 쓰레드들은 대기하게 되며 순차적으로 처리.
Global Lock
그러나, 티켓이 몇만개에 유저가 몇십만명이 접속한다면? 하나의 OS에서 처리하기에는 너무나 많은 시간이 소요
이 때 같은 티켓에 대한 Lock을 별도의 정보에 저장한 뒤, 이를 허가한 요청만 승인해준다.