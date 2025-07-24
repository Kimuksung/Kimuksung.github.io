---
layout: post
title:  "Iceberg Architecture"
author: Kimuksung
categories: [ DataLake, Iceberg ]
#blog post image
image: assets/images/iceberg.png
comments: False
featured: False
---

<br>
안녕하세요

오늘은 AWS Iceberg가 어떻게 구성되어 있는지와 Spark에서 어떻게 동작해서 데이터를 읽어오는지를 알아보겠습니다.

Iceberg를 왜 도입하게 되었는지는 도입 배경을 참고 해주세요
<br>

### Iceberg Architecture

#### Layer
1. Data Layer
2. Metadata Layer
3. Catalog Layer

<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2Fclcp3z%2FbtsPp9AfeQv%2FAAAAAAAAAAAAAAAAAAAAAJy85lze42qigxV9QBaNc6L2uLB_RC6i-WEJ209-aPSV%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1753973999%26allow_ip%3D%26allow_referer%3D%26signature%3DuOiC6NZwW93HRqsdlH505ZRgU6I%253D" width="300" />
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FJ6vho%2FbtsPrtEvGYc%2FAAAAAAAAAAAAAAAAAAAAAFjCuuIXn3pu2b8tJX44qcZHoNyrbTgiAyKQbBRThwqK%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1753973999%26allow_ip%3D%26allow_referer%3D%26signature%3DUHy8kCNVZD4meYIguACBM64m7LU%253D" width="300" />


##### 1. Data Layer
- 데이터가 저장되는 위치
- 파일 형식은 Parquet(Default), Orc, Avro 지원
- Cloud 객체 스토리지(S3), 분산 파일 시스템(HDFS) 지원

##### 2. Metadata Layer
- Iceberg 테이블 메타데이터파일
- 데이터, 메타데이터를 추적하는 트리구조
- 파일 변경 할 수 없기 때문에 insert, merge, upsert, delete 작업이 발생 시 마다 새로운 파일 생성

- **Manifest file**
  - 각 파일 대한 세부 정보, 통계, 데이터 계층 파일 위치
  - content = 0(데이터 파일), 1(위치 삭제 데이터 파일), 2(동등 삭제 파일)
  - *.avro 파일 형식 저장

- **Maifest List**
- Manifest file을 추적하는 용도
- Manifest file의 위치, 파티션, 데이터의 상한/하한
- Iceberg의 Snapshot 정보도 가지고 있음
- *.avro 파일 형식 저장

- **Metadata files**
  - *.metadata.json Manifest List 추적
  - 앞 5자리는 점진적으로 증가하여 구성
  - 특정 시점의 Iceberg Metadata
    - Table Schema
    - Partition
    - Snapshot
    - current Snapshot
    - manifest-list
    - statistic
    - metadata-log

##### Catalog Layer
- Table의 현재 Metadata file을 가리키는 Pointer
- Metadata pointer를 저장하고 atomic하게 보장만 해줄 수 있으면 가능
E.g) Aws Clue Catalog, Hive, S3, Hadoop

### Spark 동작 원리
1. Catalog로 부터 metadata file locations을 얻어냄 
2. catalog로부터 얻어낸 v1.metadata.json 파일을 읽고, schema/partition 전략을 이해 
3. write datalayer parquet 
4. write manifest file 
5. create manifest list 
6. create v2.metadata.json 및 new snapshot catalog값을 v2로 변경
