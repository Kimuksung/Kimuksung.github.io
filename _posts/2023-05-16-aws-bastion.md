---
layout: post
title:  "AWS EC2 접근 With Bastion"
author: Kimuksung
categories: [ AWS ]
tags: [ AWS, EC2, Bastion ]
# 댓글 기능
comments: False
---

안녕하세요  
오늘은 AWS EC2 Service에 접근하기 위한 방법을 알려드리려고 합니다.  
일반적으로 EC2 Service는 Private Subnet에 구성되어, 외부에서 접근이 불가능합니다.  

이를 도와주는 역할이 Bastion입니다.

<br>

##### 기본 개념

---
좋은 예시가 있어 가져와 활용하였습니다. - [참고](https://err-bzz.oopy.io/f5616e26-79ca-4167-b2eb-140de69b9b54)
- VPC = 하나의 학급
- Subnet(사설 IP) = 반 번호
- 학급 내에서 1번,2번 .. 15번.. 이 있다고 하면, 번호를 부르며 통신을 할 수 있습니다.
- 외부에서 1반의 1번을 부르고 싶다면?
- 공인IP - 고유한 번호(127.0.0.1)를 가지고 있으면 부를 수 있다.

But

- 공인 IP 갯수는 한계
- 외부에서 불려지는 것을 원치않을 수도 있다;

공인 IP가 없는 친구들을 부르기 위해 필요한 것이 `Bastion`

**문지기 한명을 설정하여 해당 문지기만 공인 IP로 연결한 뒤 다른 친구들을 불러오게 한다.**

*Bastion = 수호자,보루,요새

*Proxy = 대리인,내통자

활용 방안?

- EC2 instance 접근 시 Bastion Host를 통해서만 접근!

<br>

##### AWS Bastion Architecture
---
- Bastion Host = Public Subnet(공인 IP)
- Private subent = 공인 IP 할당 X , Bastion으로 부터만 트래픽을 받을 수 있도록 설정

![https://ifh.cc/g/CjaRMY.jpg](https://ifh.cc/g/CjaRMY.jpg)

##### Bastion 접속 
---
- [Connect SSH 처리 방법](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-connect-master-node-ssh.html)

```bash
# pem file 권한 설정
$ chmod 400 pem_name.pem

# Connect Bartion
$ ssh -i  ~/pem_name.pem ubuntu@ip
```

##### AWS EC2 접근해보기
---
- Bastion 내부에 저장된 pem directory에 접근합니다.
- EC2 Private IP 접근

```python
ssh -i pem/pem_name.pem [ubuntu@](mailto:ubuntu@ip){ec2 private ip}
```