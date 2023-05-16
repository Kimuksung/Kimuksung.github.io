---
layout: post
title:  "Aws Redshift Env&Architecture"
author: Kimuksung
categories: [ Redshift, AwS ]
tags: [ AWS, Redshift ]
image: "https://ifh.cc/g/rD7FlS.png"
# 댓글 기능
comments: False
---


##### Redshift Architecture
---
- Wclub
    - 1Leader Node
    - 3Compute Node
- Cluster
    - Database
        - Slice
            - 각각 메모리, Disk, Cpu 할당
            - 독립적인 워크로드로 병렬 실행
    - Leader Node
        - query 실행 및 데이터 분산 처리 담당
        - Client와 Communicate 담당
        - In-Memory Caching
    - Compute Node
        - 데이터 저장소
        - Slice가 1MB block의 여러 테이블로 저장
        - 데이터 저장과 실행이 같이 이루어져 전송 속도 향상
            
            ![https://miro.medium.com/max/720/1*PrjO10U-8R-Ku2YXgZ-LPA.webp](https://ifh.cc/g/R65w2l.webp)
            

- 병렬 처리
- Data Compression
- Columnar Storage
    - 칼럼을 통하여 접근하여도 Row를 찾을 수 있다.
    - 필요 데이터만 접근하여 I/O를 줄여준다.

<br>

##### 외부 연결 방법 ( IP ) with Inbound

---

- VPC 설정과 동일
- aws redshift 접속
- Cluster overview → cluster → properties → vpc security group → inbound rules → edit inbound rules

![스크린샷 2022-12-30 오전 11.57.23.png](https://ifh.cc/g/khs9JW.png)

![https://ifh.cc/g/hsA2my.jpg](https://ifh.cc/g/hsA2my.jpg)

![스크린샷 2022-12-30 오후 12.04.28.png](https://ifh.cc/g/W4QcMo.png)