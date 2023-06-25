---
layout: post
title:  "Hadoop Client Read&Write"
author: Kimuksung
categories: [ Hadoop ]
#blog post image
image: "https://i.ibb.co/DCybG93/hadoop.jpg"
comments: False
---

Client가 HDFS에 Read하고 Write하는 과정

##### Read
---
- Sample.txt = 256MB
- 2개의 Block = Block A(128M) + Block B(128M)
1. `Open()` = HDFS Instance에게 call
2. NameNode에게 Block이 어디있는지 정보 요청 → return addresss
3. file block의 Primary Data Node에게 `read()` 요청
4. data는 실시간으로 client에게 전송
5. block의 연속적인 데이터를 가져와 계속하여 전달
6. client가 데이터를 전부 받았으면 `close()`

![example](https://i.ibb.co/3mYcvbp/HDFS-Read.png)

##### Write
---
- HDFS는 1write many read라는 관점을 가지고 있다는 점을 이해해야한다.
1. DFS에 open() 호출
2. Name Node에 Create() 호출하면서, 파일이 이미 존재하는지, 유저가 파일 권한이 있는지 체크
3. hdfs는 데이터를 여러 곳에 써야하는데, packet을 분할해야한다. Data Queue를 통하여 연결할 DataNode를 DataNode에게 물어보고 승인 받은 뒤 알려준다. 3,4,5번을 연결하고 이후에 파이프라인을 통해서 마지막 DataNode에 까지 전달
4. Data Node의 ACK를 기다려 승인을 받는다.
5. 모든 replicas에서 승인을 받게 되었다면 FDSData OutputStream의 연결 종료


![example](https://i.ibb.co/wcJ1B8v/HDFS-write-1.png)