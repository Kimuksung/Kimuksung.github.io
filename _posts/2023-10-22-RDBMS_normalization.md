---
layout: post
title:  "[RDBMS] 정규화&역정규화"
author: Kimuksung
categories: [ RDBMS ]
tags: [ RDBMS, Normalization, DeNormalization]
image: assets/images/rdbms.png
# 댓글 기능
comments: False
---

##### 정규화(Normalization)
---
<aside>
💡 RDBMS에서 중복을 최소화 및 종속 관계의 속성을 제거하기 위한 프로세스
</aside>

- 데이터의 속성끼리 종속 관계를 분석하여 여러 개의 릴레이션으로 분해
- 기본 정규형 - 1,2,3 정규화와 보이스/코드 정규화
- 고급 정규형 - 4,5 정규화

- OLTP 데이터베이스는 실 서비스 **CRUD**가 많이 일어나기 때문에 **정규화** 되는 것이 좋다.
- OLAP는 분석, 리포팅용이기 때문에 **연산의 속도**를 위해 **역정규화**하기도 한다. ( 특히, 대량의 select 처리가 중요한 경우 )

장점
- 중복된 데이터는 많은 문제를 일으키기 때문에 이상 현상 발생 가능성을 줄인다.

단점
- 릴레이션 분해로 인해 Join 연산이 많아진다.

이로 인해 응답시간이 증가할 수도 있다. (무조건 증가하는 것은 아니다.)

<br>

##### 1정규형
---
- 같은 성격과 내용의 칼럼이 연속적으로 나타내는 컬럼이 존재 할 때 PK를 추가해새로운 테이블을 생성하여 기존 테이블과 M:N 관계에서 **`1:N`**로 변경
- 음식점과 음식점에서 판매하는 내역을 구성하려고 한다고 해보자.
- 여기서 판매상품은 여러개의 상품으로 구성되어있으며, 판매 상품은 음식점과 별도의 성격을 가지는 유형이니 분리하여 준다.

<a href="https://ibb.co/FsHsq8v"><img src="https://i.ibb.co/wJcJrpH/image.png" alt="image" border="0" width=1500></a>

<a href="https://ibb.co/RcDp2b1"><img src="https://i.ibb.co/XtCZXL9/1.png" alt="1" border="0" width=1500></a>


<br>

##### 2정규형
---
- 1정규화를 마친 구조를 봐보자.
- 가맹점id, 위치, 이름, 테이블 번호에서 중복이 발생한다.
- 주문번호, 주문 시각, 판매수량, 판매id에 필드는 기존과 다르게 나누어 준다.
- 가맹점id를 외래키로 등록하여 이를 연결시켜 준다.
- 이렇게 구성하면 중복 없이 각 테이블 별로 나눌 수 있다.

<a href="https://ibb.co/hKqhZyL"><img src="https://i.ibb.co/wKv3wRB/2.png" alt="2" border="0" width=1500></a>

<br>

##### 3정규형
---
- 테이블의 키가 아닌 칼럼들은 PK에 의존해야 한다.
- 이전적 함수 종속 관계 : 실제로는 PK가 아닌 다른 칼럼에 의존하는 칼럼이 존재 할 수 있다.
- 가맹점 정보에서는 가맹점id에 의존하는 칼럼들이 존재해야한다.
- 테이블 별 주문 정보에서는 주문번호에 의존하는 칼럼들이 존재해야한다.
- 여기서 테이블 번호가 주문번호에 의존하는 칼럼이라고 볼 수 없기는 하지만, 그렇다고 다른 일반 컬럼에 의존하지도 않기에 건들이지 않았다.
- 만약, 의존하는 칼럼이 존재한다면 별도의 테이블을 따로 구성하여 FK로 연결시켜 준다.

<a href="https://ibb.co/6JMchws"><img src="https://i.ibb.co/wrkZHg6/3.png" alt="3" border="0" width=1500></a>

<br>

##### 역정규화
---
<aside>
💡 성능, 편의성을 위해 정규화 작업을 되돌린다. (Join 비용과 select 간소화)
</aside>

- 위와 같은 테이블 설계 구조로 실서비스에 반영했다고 가정해본다.
- 가맹점이 천만개 있다고 가정하고 가맹점 별 매출과 최다 판매 정보 지표로 보내주어야한다고 해보자.
- Join문과 Group by sum을 활용해서 구성하기에 쿼리 자체가 오래 걸릴 것이다.
- 추가적으로, 지표를 보내주기 위한 가맹점 번호가 필요할텐데 그렇다면 가맹점 정보, 주문 정보, 판매 정보 3개의 테이블을 `**Join**` 시켜 주어야 한다. 물론 외래키를 지정한다면, 속도는 빨라지겠지만 Join 연산 자체가 추가 되기 때문에 오래 걸릴 것이다.
- 테이블 자체에 해당 값이 존재한다면, `**불필요한 연산을 줄일 수 있다.**`

- 여기서 성능을 더 개선시킬 방법은 없을까?
- 가맹점 별로 순차적으로 보낸다고 가정하면 위에서 역정규화한 테이블을 가맹점 별 ID 별로 파티셔닝을 하여 두고 접근하면 부하를 줄일 수 있지 않을까? 라는 생각이 든다.
- Mysql에서는 이를 위해 **파티션 기법**을 지원한다.

<a href="https://ibb.co/JqvKqVx"><img src="https://i.ibb.co/bFsPFTH/image.png" alt="image" border="0" width=1500></a>

단점
데이터 Update, Insert 비용이 높다.
데이터의 무결성을 해치기 때문에 주의해야한다. → 데이터 이상 현상이 발생할 가능성이 높다.
데이터 중복 저장으로 인해 추가 저장공간 확보가 필요하다.

아직 실서비스에서 역정규화를 적용해본 사례는 없다.. 그 만큼 많은 양의 데이터를 처리해본적이 없기 때문에 나중에 까먹지 말고 적용시켜볼 수 있도록 해야겠다.