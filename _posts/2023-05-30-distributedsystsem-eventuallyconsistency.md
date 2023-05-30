---
layout: post
title:  "Distributed System - Eventually consistsency&Strong Eventuall consistency"
author: Kimuksung
categories: [ Distributed System ]
tags: [ Eventually consistsency, Strong Eventuall consistency ]
image: "https://ifh.cc/g/sCSX4x.png"
comments: False
---

앞에서 공유드린, BASE, CAP를 알고 있다는 가정하에 추가적으로 설명합니다.

##### Eventually Consistency
---
- 최종 일관성이라는 의미
- 데이터의 이용률을 높인다.
- 전세계적으로 각 대륙별 AZ가 있다고 하여보자, 이 AZ 지역 안에서도 multi node를 구성하고 있다.
- 유럽에서 A라는 사람이 글을 썼을 때 다른 AZ들의 Node들에 전달이 되어야 한국에 있는 사람이 해당 글을 보기 위해서는 ?
- 모든 노드에 배포되기 전까지는 불일치하는 데이터를 볼 수 있다. → 이로 인해 다른 블로그를 보게 될 수 있다.
- 이 때 언젠가 전달되어 동기화 처리 될 것이라고 믿는 것이 최종 일관성

![image](https://ifh.cc/g/15K5vf.jpg)

##### Strong Eventual Consistency
---
- 강한 일관성으로 분산처리 시스템에서 Consistency를 선택하는 경우
- 모든 사용자들에게 동일한 데이터를 제공
- 전세계적으로 각 대륙별 AZ가 있다고 하여보자, 이 AZ 지역 안에서도 multi node를 구성하고 있다.
- 유럽에서 A라는 사람이 글을 썼을 때 한국에 있는 사람이 글 쓴 내용을 동일하게 보게 하기 위함
- 대신, Lock을 잡고 있어 다른 노드들에게 전송이 잘 되었는지 파악을 계속하여 확인

![image](https://ifh.cc/g/KqztQh.jpg)

##### 추가적인 의문 사항 정리
---
- **`Write 속도`** = 노드 간 복제 동기화를 기다릴 필요가 없어, write 속도가 강한 일관성 보다 빠르다.
- **`Read 속도`** = 최종 일관성은 강한 일관성보다 대부분 Read가 빠르다. 일반적으로는 동기화를 기다릴 필요가 없기 때문에 다만, 노드 간의 복제본의 수,네트워크 지연, AZ와 같은 위치에 따라 달라질 수 있다.
- 최종 일관성, 강한 일관성은 분산 처리 기반 특성 **`BASE`**를 가지고 있다.
- 최종 일관성은 **동일한 결과를 던져주지 않을 수 있다**.

참조
- https://www.acodersjourney.com/eventual-consistency/