---
layout: post
title:  "Nosql Preperties"
author: Kimuksung
categories: [ Nosql ]
tags: [ Mongodb ]
image: "https://ifh.cc/g/WAA71Q.png"
comments: False
---


##### Nosql 특징
---
- 유연한 스키마 - 데이터 구조가 자주 변경될 것으로 예상
- Read,Write 속도가 빠르다.
- 다양한 데이터 모델 - Key,value / 문서 / 칼럼 패밀리/ 그래픽 Database 지원
- 복제와 분할 허용 - Application에 맞게 스토리지를 최적화 많은 트래픽 부하를 감당
- 대규모 데이터 처리 - 대규모의 데이터 세트를 처리하고 저장하는데 적합

##### 유연한 스키마
---
- 다양한 빅데이터 구조를 처리하기 위하여 자주 변경되는 구조에도 문제 없이 적재 - RDBMS에서는 스키마구조 정의가 필요하다

##### Read,Write 속도
---
- 일반적으로 Key-value, Document 기반 데이터 모델로 SQL 쿼리보다 빠른 읽기,쓰기 제공 ( 경우에 따라 성립이 안 할 수 있다 )
- **분산 처리** = 여러 노드에 분산시키는 분산 처리 지원 → 데이터를 병렬로 읽거나 쓸수 있다.
- **데이터 불변성** 없음 = Transaction을 지원, 이를 통해 Lock 존재 → 오버헤드 및 읽기,쓰기 딜레이
- **스키마가 존재하지 않는다**. - 데이터 유형을 즉시 저장 및 스키마 변경 프로세스가 필요없다.
- **데이터 복제,샤딩** = 데이터를 여러 노드에 복제,샤딩하여 빠르게 접근 가능

- 왜 key,value document가 더 빠를까?
    - Join, 변환 쿼리 없이 데이터 유형,속성 추가 가능

##### Replica와 Sharding
---
- Replica
    - Data를 다른 노드에 복사하여 가용성, 읽기 성능 향상
    - 데이터 일관성 처리가 관건
- Sharding
    - 데이터를 수평적으로 분할하여 각 샤드에 데이터의 일부를 저장
    - 여러 샤드에 걸쳐 데이터를 분산 정하여 확장성도 높인다.
    - 한번에 여러 노드에서 Write를 처리하기 때문에 속도 향상
    

RDBMS와 Replica, Sharding 차이
- RDBMS에서는 Transaction이 존재하기 때문에 Replica시 Overhead가 크다.
- RDBMS에서는 샤딩이 가능하나 구현이 복잡하고 특징에 알맞지 않다.

<br>

##### 대규모 데이터 셋 특화
---
- 앞에선 본 내용들을 종합하여 보면, 복제를 활용하여 Read 속도도 향상
- 샤딩을 통해서 Write 속도도 향상
- 스키마 구조가 정해져 있지 않고 마음대로 Key-value, 문서 형태로 저장하면 되기 때문에 바로바로 처리 가능

위와 같은 이유로 빅데이터 처리와 분산 처리 시스템에서 Nosql을 무조건 언급 할 수 밖에 없나 보다..