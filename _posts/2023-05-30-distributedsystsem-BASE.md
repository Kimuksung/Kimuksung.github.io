---
layout: post
title:  "Distributed System - BASE"
author: Kimuksung
categories: [ Distributed System ]
tags: [ BASE ]
image: "https://ifh.cc/g/sCSX4x.png"
comments: False
---


분산처리에 들어가기 앞서, 기본 특성을 살펴봅니다.

##### BASE
---

- 분산 처리 시스템의 기본
- RDBMS의 ACID와 대조적으로 가용성, 성능을 중시

**B**asically **A**vailable

- 기본적으로 가용성을 중시
- Optimize locking 및 Q 사용
- 부분적인 고장이 있을 수 있나, 나머지가 사용 가능

**S**oft-state

- 사용자가 refresh, modify 하지 않으면 데이터가 expire
- node 상태는 외부에서 전송된 정보를 통해 결정
- 분산 처리된 노드에 데이터가 업데이튼 되는 시점 = 데이터가 노드가 도달 시 ( 최신 상태 데이터로 overwrite )

**E**ventually consistency

- 네트워크, 시스템 부하와 같은 이슈가 있더라도 기다리다보면 언젠가 분산 노드들에게도 데이터가 반영

##### Optimize lock

---

- Database 충돌 상황을 개선할 수 있는 방법
- select → update와 같은 구문에서 문제 발생
- 값을 먼저 변경하겠다고 선언 후 수정하여 동일한 조건으로 값을 수정 할 수 없게한다.
- Transaction이 존재하지 않는다.

Rollback

- 별도의 Transaction을 잡지 않는다.
- 충돌이 발생하.  수정이 되지 않는다면, App level에서 수동으로 처리해줘야한다.

![image](https://ifh.cc/g/GBrdno.png)

##### perssimitic lock

---

- RDBMS에서 Repeatable, Serializable의 격리정 수준
- Transaction이 시작될 때, Shared Lock or Exclusive Lock을 걸고 시작.
- Write 시 Exclusive Lock이 설정되는데, 이 때 다른 세션에서 Select,Update 처리 실행 중이라면 Exclusive Lock을 할당 받을 수 없어 대기 후 Lock을 획득하면 Transaction 시작

Rollback

- RDBMS의 ACID에서 배운것과 동일
- Transaction이 실행되고 난 뒤의 모든 값들을 되돌려준다.

![image](https://ifh.cc/g/zVgBmH.png)

위의 경우로 인하여 **Distributed System에서는 CAP이론 3가지를 모두 만족하는 것이 불가능함**

- Consistency : 모든 Node 데이터는 어떤 순간에도 항상 같은 데이터를 소유
- Availability : 분산 시스템은 모든 Request에 대해 성공/실패를 언제든지 Response 할 수 있다.
- Partition Tolerlance = 네트워크 장애, 서버 부하 등 여러 이슈에도 시스템은 동작한다.

참조
- https://sabarada.tistory.com/175