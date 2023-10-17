---
layout: post
title:  "AWS Private subnet (VPC, NAT gateway, Bastion)"
author: Kimuksung
categories: [ AWS ]
tags: [ AWS, VPC, NAT, Bastion]
image: assets/images/bastion.png
# 댓글 기능
comments: False
---


오늘은 NAT Gateway, Bastion이 필요한 상황을 알아보고 AWS 네크워크 환경을 구성해볼 예정입니다.

##### VPC, Subnet 구성
---
- VPC, Subnet을 먼저 생성하여 줍니다.
    <a href="https://ibb.co/Twt90w7"><img src="https://i.ibb.co/HKq5PKb/Untitled-52.png" alt="Untitled-52" border="0"></a>    

    <a href="https://ibb.co/Dbnf4B0"><img src="https://i.ibb.co/CtFKBjR/Untitled-53.png" alt="Untitled-53" border="0"></a>
    
- 구성이 완료되었다면 아래 그림과 같이 설정됩니다.
    <a href="https://ibb.co/30y9DCh"><img src="https://i.ibb.co/LCzXLZp/Untitled-54.png" alt="Untitled-54" border="0"></a>
    

- 라우팅에 IGW를 연결하여 주면 Public Subnet 입니다.
- IGW를 생성하고 연결하여 줍니다.    
    <a href="https://ibb.co/4F676Lq"><img src="https://i.ibb.co/GdK0KD6/Untitled-55.png" alt="Untitled-55" border="0"></a>

- 라우팅 테이블 설정을 다시 하여줍니다.    
    <a href="https://ibb.co/WncWPhg"><img src="https://i.ibb.co/H7rDxSn/Untitled-56.png" alt="Untitled-56" border="0"></a>

    <a href="https://ibb.co/54FDzVP"><img src="https://i.ibb.co/B3ND7R9/Untitled-57.png" alt="Untitled-57" border="0"></a>

    <a href="https://ibb.co/X4JQ2fK"><img src="https://i.ibb.co/rtyj7Kz/Untitled-58.png" alt="Untitled-58" border="0"></a>

    <a href="https://ibb.co/mXbYBzW"><img src="https://i.ibb.co/1d7pn2W/Untitled-59.png" alt="Untitled-59" border="0"></a>

    <a href="https://ibb.co/Gn8bkQ1"><img src="https://i.ibb.co/djVz643/Untitled-60.png" alt="Untitled-60" border="0"></a>

    <a href="https://ibb.co/R3JFj9J"><img src="https://i.ibb.co/K28PKX8/Untitled-61.png" alt="Untitled-61" border="0"></a>
    
<br>

##### Bastion Instance란?
---
- Private Subnet에 인스턴스를 구성하여, 외부 서비스에 접근하지 못하도록 구성하는 경우에 사용합니다. ( 내부 백오피스 )
- VPC내 Private Subnet에 바로 접근이 불가능하기에 Bastion(Public Subnet)을 인스턴스로 구성하여 접속 후 다시 같은 VPC에 Private IP로 접속하는 개념입니다.
- 인스턴스를 구성하였다면, SSH 연결을 위해 Security group inbound ssh 를 추가하여 줍니다.
    <a href="https://ibb.co/PzbKvc5"><img src="https://i.ibb.co/qF4K3NR/2023-10-17-2-47-03.png" alt="2023-10-17-2-47-03" border="0"></a> 

- Public Subnet에 임의의 인스턴스(Bastion)를 띄워줍니다.
<a href="https://ibb.co/Hg0HkzV"><img src="https://i.ibb.co/h8kmtWy/Untitled-62.png" alt="Untitled-62" border="0"></a>
        
- Private Instance에 다이렉트로 접속하려고 하면 다음과 같이 에러가 발생합니다.
    <a href="https://ibb.co/GsqL2zf"><img src="https://i.ibb.co/5YPy8bX/Untitled-63.png" alt="Untitled-63" border="0"></a>
    
- Bastion을 통하여 우회 접속해보겠습니다.
- SSH 파일이 필요하기에 업로드하여 줍니다.
    
    ```python
    $ scp -i pemfilename.pem filename user@public-ip:~/file_directory
    ```
    
    <a href="https://ibb.co/DY0NsDK"><img src="https://i.ibb.co/42hc9ZN/Untitled-64.png" alt="Untitled-64" border="0"></a>
    
<br>

##### Private Subnet에 구성된 Instance를 외부와 연결하려면 어떻게 해야할까?
---
- Private Subnet의 인스턴스는 외부로 나갈 수 있는 루트가 막혀있다.
    <a href="https://ibb.co/ncWC7jy"><img src="https://i.ibb.co/3NJzpcb/Untitled-65.png" alt="Untitled-65" border="0"></a>
    
    
- `NAT GATEWAY` 로 이를 해결 가능하다.
- Private 인스턴스들만 외부와 통신하고 싶은 경우 사용한다. ( 인터넷 연결로 라이브러리 다운, 일방적인 데이터 요청 .. )
- NAT Gateway를 Public subnet으로 만들어 통신 길을 만들어준다. ( 대다수의 블로그에서 NAT Gateway는 Public subnet이라고만 써 있는데, 사용 용도가 외부로 연결하기 위해서라는 의미로 해석)
    <a href="https://ibb.co/VvngcP2"><img src="https://i.ibb.co/Wc8k4Yg/Untitled-66.png" alt="Untitled-66" border="0"></a>
    
- 이 때, 외부와 통신하기 위해서는 고정 IP가 필수적인데 이를 `Elastic IP`로 설정하여 준다.
    <a href="https://ibb.co/wBNTFLb"><img src="https://i.ibb.co/HBD0f2M/Untitled-67.png" alt="Untitled-67" border="0"></a>
    
- 위 작업이 끝나면, Private Subnet과 연결된 라우터에 NAT가 존재하는 Subnet에 이를 연결시켜 준다.
<a href="https://ibb.co/cY30SJt"><img src="https://i.ibb.co/CP8g3Bv/Untitled-68.png" alt="Untitled-68" border="0"></a>
    
    <a href="https://ibb.co/3RWfNnX"><img src="https://i.ibb.co/Y08Dkx9/Untitled-69.png" alt="Untitled-69" border="0"></a>

