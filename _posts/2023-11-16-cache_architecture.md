---
layout: post
title:  "캐싱 아키텍처"
author: Kimuksung
categories: [ Cache, Redis ]
tags: [ Cache, Redis ]
image: assets/images/redis.png
comments: False
featured: True
---

안녕하세요.  
오늘은 캐싱 전략에 대해 알아보려고 합니다.  

알아보는 이유는 최근 하나의 게시판에 엄청 많은 트래픽이 몰렸을 경우 게시판을 보는 사용자 수를 어떻게 처리하면 더 효율적일 수 있을까? 라는 질문으로 부터 시작하였습니다.  
예시) 유명인, 이벤트, 티켓팅 정보 게시판  
  
Database 입장에서 Select/Update/Insert Transaction 시 동시에 처리가 가능할까?  
DB 구조를 id/ 게시판 id/ 게시판 제목/ 게시판 내용/ 작성자id/ 좋아요 수/ 글 조회 수/ 현재 글을 보고 있는 사람 수라고 해보겠습니다.  
게시판을 바라보고 있는 유저 수를 카운트해야한다고 할 때 -> 게시판 수정 및 현재 게시판을 보는 유저들을 세기 위해 update 처리가 동시에 실행되며 lock이 걸릴 것이다.  
  
그러면 RDS에서 Read/Write 인스턴스를 분리하는 작업을 한다고 해서 해결이 가능할까?  
-> Read를 줄여 부하를 줄일 수는 있지만, 근본적인 lock 문제는 해결되지 않을 것이다.  

일단 좋아요 수/ 글 조회 수 같이 카운트 하는 부분을 분리해야 할 거 같다.  
id/ 게시판 id/ 게시판 제목/ 게시판 내용/ 작성자 id  
게시판 id/ 좋아요 수/글 조회 수/ 현재 글 바라보는 유저 수  
  
문득, 인스타/링크드인 같은 회사에서는 동일한 문제를 겪었을 텐데 어떤 방식으로 해결하였을까? 라는 궁금증으로 찾아보았다.  
"대용량 트래픽 처리를 위한 대규모 시스템 설계하기"라는 글에서 아래와 같은 정보를 얻었다.  
현재까지 진화해온 설계 방식이라고 한다.  
1. 수직적 Scale-up
- User -> Server -> DB
2. 수평적 Scale-up
- User -> LoadBalancer -> WAS -> DB
3. Database 이중화
- User -> WAS -> DB(Master-Replica)
4. 메모리 DB
- User -> WAS -> Memory DB -> DB(Master-Replica)
5. MSA
- User -> Domain(WAS -> Memory DB -> DB(Master-Replica))

힌트를 얻고 메모리 DB에 대해 상세히 알아보았다.  

<br>

#### 캐싱 아키텍처
- 데이터 베이스 부하를 줄이고 시스템의 성능 향상을 시킬 수 있는 주요 전략 중 하나
- 캐시 = 메모리를 사용하여 빠르게 데이터를 응답하여 사용자에게 빠르게 서비스 제공

#### Redis
---
- **Server Side** 에서 캐싱을 구성
- User → App → Server → Redis → Database

##### Read 전략
---
- 1) Inline Cache
    - Batch Job → DB → Redis
    - App → Redis로 접근하여 캐시 데이터가 있다면 종료
    - 없다면, App → DB

<br>

- 2) Cache-Aside
    - **`유저`** 가 주체가 되어 **`Cache`**시키는 형태
    - App → Redis (캐싱 여부 확인)
    - 없다면, DB에서 조회
    - DB 조회 결과를 Redis에 반영
    <br>
    <a href="https://ibb.co/w7FGpZ5"><img src="https://i.ibb.co/zFT9PWq/cache-aside.png" alt="cache-aside" border="0"></a>
        
<br>

- 3) Read-Through
    - **`Redis`**가 주체
    - App → Redis (캐싱 여부 확인)
    - Cass miss → Redis → 직접 DB 조회 하여 반영

    <br>

    <a href="https://ibb.co/vsmjsCx"><img src="https://i.ibb.co/n3fL3qs/read-through.png" alt="read-through" border="0"></a>
    
<br>

##### Write 전략
---
- 1) Write-Through
    - Redis Cache에 데이터를 추가/업데이트
    - App → Redis ( **`Sync`** )
    - Redis → DB 순으로 Write 하기에 latency 발생
    - 캐시와 DB는 항상 일관성을 유지
    - 캐시 히트 확률이 높다.
    - 쓰지 않는 데이터도 Cache 될 수 있다. → 보완 : TTL 전략을 사용
    - E.g) DynamoDB Accelerator

    <br>

    <a href="https://ibb.co/1ZyZQBz"><img src="https://i.ibb.co/D4c4Wmr/write-through.png" alt="write-through" border="0"></a>
        
<br>
    
- 2) Write-Around
    - 데이터는 DB에 직접 저장
    - 읽은 데이터만 Cache에 저장
    - 자주 읽지 않는 데이터에 대해서는 Resource 감소
    - 캐시 데이터를 건들지 않는다.
    - 캐시와 디비랑 다를 수 있다.
    - heavy read application에서는 용이
    - 새로운 데이터 읽기에서는 항상 cache-miss 발생
    - DB가 장애가 발생 시 write는 실패
    - E.g) Log/ 채팅  
    <br>
    <a href="https://ibb.co/MBtY6b8"><img src="https://i.ibb.co/TW7VLZK/2023-11-16-2-28-06.png" alt="2023-11-16-2-28-06" border="0"></a>
    
<br>

- 3) Write-Back / Write-Behind
    - App → Redis
    - Redis → DB ( `**Async**` )
    - Redis → DB 옮겨가는 과정을 주기적인 batch 작업으로 실행하기에 load, cost 감소
    - Write가 빈번하게 발생하는 작업에 적합
    - Read-Through 와 결합하여 최근 업데이트 + 액세스 데이터 사용 시 적합
    - DB Write를 줄일 수 있다.
    - Heavy Write 작업에 효과적
    - DB가 장애가 발생해도 write 작업은 동작한다.
    - 캐시에만 write가 되고 db에 write 처리 되지 않는다면, 이슈가 발생 ( Redis 또한 Master-Slave 형태의 구조를 구성 )
    - E.g) 조회 수/좋아요 카운트 →캐시 → 일정 시간 마다 DB 반영  
    <br>

    <a href="https://ibb.co/KDrVr1F"><img src="https://i.ibb.co/cwr2rmD/write-back.png" alt="write-back" border="0"></a>
    
    <a href="https://ibb.co/m53d2Dz"><img src="https://i.ibb.co/GQqG8xP/2023-11-16-2-35-38.png" alt="2023-11-16-2-35-38" border="0"></a><br /><a target='_blank' href='https://usefulwebtool.com/'></a><br />

    
##### 참조
---
- https://crazy-horse.tistory.com/139
- https://medium.com/bliblidotcom-techblog/redis-cache-aside-pattern-72fff2e4f927#id_token=eyJhbGciOiJSUzI1NiIsImtpZCI6IjViMzcwNjk2MGUzZTYwMDI0YTI2NTVlNzhjZmE2M2Y4N2M5N2QzMDkiLCJ0eXAiOiJKV1QifQ.eyJpc3MiOiJodHRwczovL2FjY291bnRzLmdvb2dsZS5jb20iLCJhenAiOiIyMTYyOTYwMzU4MzQtazFrNnFlMDYwczJ0cDJhMmphbTRsamRjbXMwMHN0dGcuYXBwcy5nb29nbGV1c2VyY29udGVudC5jb20iLCJhdWQiOiIyMTYyOTYwMzU4MzQtazFrNnFlMDYwczJ0cDJhMmphbTRsamRjbXMwMHN0dGcuYXBwcy5nb29nbGV1c2VyY29udGVudC5jb20iLCJzdWIiOiIxMTA5NTM0MzEzNjk0NTg4Nzg1MzgiLCJoZCI6IndjbHViLmNvLmtyIiwiZW1haWwiOiJrYWkua2ltQHdjbHViLmNvLmtyIiwiZW1haWxfdmVyaWZpZWQiOnRydWUsIm5iZiI6MTcwMDExMDEyMywibmFtZSI6IkthaSBLaW0iLCJwaWN0dXJlIjoiaHR0cHM6Ly9saDMuZ29vZ2xldXNlcmNvbnRlbnQuY29tL2EvQUNnOG9jTHMyNVIyaENqRXY0c2lwVnRScXk0M2NMVlRJQ0xHQ3lfaVR6MHktUzhEPXM5Ni1jIiwiZ2l2ZW5fbmFtZSI6IkthaSIsImZhbWlseV9uYW1lIjoiS2ltIiwibG9jYWxlIjoia28iLCJpYXQiOjE3MDAxMTA0MjMsImV4cCI6MTcwMDExNDAyMywianRpIjoiYzU5NDAxMmI2ZGIwMjZiZGY5NzViYzIxNzJiODI4MmE4ZGI0YTgzZSJ9.a7iJrmSr0cudWyc0ErMvpxVtoD-qzGnmH6NFIegYfsamy5I8joVpOwF-mdLXHe0Eo3v4GZsikjDVd6AjiNWb9I-1QjekBpMOUeApszcR1Z5jYCmOxDy6zO7xYMeFa7cf07kW7pW9evHstV8jxj7S_W5BjaktJ2XTk_z0Q2xQ06T-D7Ciq9cB5gtyGHJ1UmH3rZvBS7UrIGQ754r8UMmKZU9snSV4M_Nb09oT0QFLo0ien9CGQhx_0LyyGdbI87WNlDKc7CwO8pPlzzEpJSHmpa-M04qBtPAsCmKFEf_iSe-PzmHLNEKTX3S7igtVa5cBsXpJey9G5l3b4TS7PYpJZw
- https://codeahoy.com/2017/08/11/caching-strategies-and-how-to-choose-the-right-one/
- https://www.youtube.com/watch?v=RtOyBwBICRs