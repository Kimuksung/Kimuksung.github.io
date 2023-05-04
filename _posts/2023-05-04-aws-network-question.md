---
layout: post
title:  "AWS Network"
author: Kimuksung
categories: [ AWS , Network ]
tags: [ AWS, Loadbalancer , ALB, NLB, IGW, NGW, Public IP, Private IP  ]

# 댓글 기능
comments: False
---

안녕하세요  
오늘은 AWS Network를 구성하면서 모르는 부분이나 상세히 이해가 안간 부분에 다시 알아보려고 합니다.

##### AWS Network 궁금한 사항들 정리

##### 1. IGW vs NGW
---
###### Internet Gateway (IGW)
- `allows instances with 'public IPs' to access the internet`
- VPC와 인터넷을 연결하여 준다.
- 정확하게는, Route table에 추가하여 public subnet과 연결 
- IGW가 없다면, VPC는 외부 Internet과 연결이 불가능하다.
- IP4, IP6 지원
- 네트워크 트래픽에 대한 가용성이나 대역폭에 영향을 주지 않는다.

###### NAT Gateway (NGW)
- `allows instances with 'no public IPs' to access the internet.`
- NAT 서비스 관리
- IGW와 유사하지만, private subnet에 있는 service를 외부 네트워크와 연결 시 사용
- Public subnet - Private subent을 연결
- IP4, IP6 지원 및 TCP, UDP, and ICMP 프로토콜 지원
- 여러 AZ영역이 있다면, 하나의 NGW로 구성 가능
- 독립적인 AZ영역으로 설정하려면, 각 AZ마다 NGW로 생성해야 한다.
- NGW는 Instance의 IP 주소를 NGW에 맞추어 변경 처리.
<a href='https://ifh.cc/v-CvJ2PA' target='_blank'><img src='https://ifh.cc/g/CvJ2PA.jpg' border='0'></a>

<br>

##### 2. NACL vs Security Group
---
###### Network ACL(Access Control List)
- `네트워크 방화벽` - 정책을 설정하여 유입되는 트래픽 제어
- **Subnet 단위로 적용**
- Subnet 모든 Instance에 적용
- 1 VPC - 최대 200개 NACL
- 1 NACL - 20 inbound + 20 outbound
- 여러 Subnet에서 같은 Subnet 사용 가능
- Subnet은 하나의 NACL만 적용 가능

###### Security Group(SG)
- `Instance 방화벽`
- **Instance 단위로 적용**
- 1 VPC - 최대 2500개 SG
- 1 SG - 60 inbound + 60 outbound

###### Subnet Connection
- 동일 Subnet - SG 정책
- 다른 Subnet - NACL -> SG 정책

<a href='https://ifh.cc/v-4aa9vZ' target='_blank'><img src='https://ifh.cc/g/4aa9vZ.png' border='0'></a>

<br>

##### 3. Application Load Balancer(ALB)
---
- Network layer -> App layer
- Http,Https Trafic 라우팅 - Host,Path 설정
- E.g) 온라인 쇼핑몰 웹 애플리케이션 -> ALB = 고객이 접속한 URL 경로(예: /products, /checkout 등)에 따라 요청을 적절한 서버 그룹으로 라우팅합니다.
- 최소 2개 이상의 AZ영역이 구성되어야 사용 가능
- SPOF 방지 / AZ 사용률 최적화 / 서비스 Instance가 증가하여도 쉽게 추가 / 문제 발생 시 하나의 AZ에서 유지보수 가능 
- Protocols: HTTP, HTTPS
- Target Types: Instance, IP, Lambda

<br>

##### 4. Network Load Balancer(NLB)
---
- Layer 4계층, App layer Level은 무시
- E.g) 데이터베이스 레플리케이션 - 데이터베이스 인스턴스에 고르게 요청을 분산,전송
- Protocols: TLS, TCP, UDP, TCP_UDP
- Target Types: Instance, IP, ALB

<br>

##### 5. Public IP vs Private IP
---
###### Public IP = External IP
- 인터넷에 다이렉트로 접근 시 사용되는 IP
- 라우터 → ISP를 통해 할당 받는다.

###### Private IP = Local IP
- 개인 장비는 라우터의 Public IP를 통해 Private IP를 할당 받는다.
- 각 장비는 unique한 ip 주소를 할당
- 회사,개인이 보안을 위해 사용 ( 같은 public ip에 있다하더라도 다른 집의 프린터를 연결 불가 )
<a href='https://ifh.cc/v-zofxob' target='_blank'><img src='https://ifh.cc/g/zofxob.png' border='0'></a>

<br>