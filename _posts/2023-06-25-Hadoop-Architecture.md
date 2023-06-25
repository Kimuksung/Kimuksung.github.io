---
layout: post
title:  "Hadoop NameNode-DataNode-Rack"
author: Kimuksung
categories: [ Hadoop ]
#blog post image
image: "https://i.ibb.co/x5f5dqQ/hdfs-components-namenode-datanode-datanode.png"
comments: False
---

##### HDFS
---
- Hadoop은 분산 파일 시스템으로 구성(Fault-toerlant 방지)
- Master/Slave Architecture로 구성
- 데이터 block은 3개의 replicas data block응로 나누어지며, rack의 여러 노드에 저장된다.

##### NameNode
---
- 역할 = **`Master`** Node(여왕벌)
    - DataNode를 관리
    - Data Block Metadata 유지보수
        
        Client 접근을 Control하는 곳
        
- secondary Name Node
    - single point of failure를 방지하기 위함
    - 현재의 fsimage + namenode log를 편집
    - HDFS version 2부터 고가용성을 위해 나온 개념
    - 즉, Main Name Node가 죽으면 Secondary가 Main 역할을 대신한다.

- Zookeeper
    - zookeeper의 daemon이 계속하여 NameNode에 health check
    - 장애를 탐지하여 영향을 최소화해주는 역할

- file metadata
    - file name
    - file permissions
    - ids
    - location
    - number of replicas
    - fsimage

##### DataNode
---
- 역할 = **`Slave`** Node(일벌)
    - 데이터 노드는 Data Block을 처리하고 저장하는 곳
    - 기본적으로 `3개의 Replica`를 각각의 Data Node 분산 저장 ( rack 인지 정책)
    - 같은 rack에서는 동일한 data block이 존재할 수 없다.
    - 하나의 Rack = N개 Data Node
- Name Node와 통신하며, 명령을 받아들여 실행( 추가 replicas 생성, 제거, data block 줄이기.. )
- 업무를 마치면 Name Node에게 보고, 한시간에 한번씩 health check

##### Rack
---
- 하나의 큰 캐비넷으로 여러 서랍장(DataNode)를 구성한다.
- Rack마다 고유의 id를 부여 관리

![](https://i.ibb.co/x5f5dqQ/hdfs-components-namenode-datanode-datanode.png)
![](https://i.ibb.co/k9XK0SM/hdfsarchitecture.png)


###### 참조
---
- https://phoenixnap.com/kb/apache-hadoop-architecture-explained
- https://dzone.com/articles/an-introduction-to-hdfs