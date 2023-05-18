---
layout: post
title:  "SSH"
author: Kimuksung
categories: [ SSH ]
tags: [ SSH ]
# 댓글 기능
comments: False
---

안녕하세요.  
오늘은 SSH에 대해서 알아보겠습니다.  
또한, Ubuntu 버전을 올리며 SSH 버전이 올라가 생긴 이슈 처리에 대해 공유드리려고 합니다.  

<br>

##### SSH란?
---
- Secure Shell의 줄임말
- 원격 호스트에 접속하기 위해 사용되는 보안 프로토콜
- 기존 Telnet이라는 원격 접속을 사용하였으나, 암호화 처리가 되지 않아 보안에 취약
- 이에 대응하기 위해 암호화를 가진 SSH 기술이 등장

E.g) Pem을 활용한 Cloud Service(AWS.. ) 등에 필수 요소

```python
ssh -i "test.pem" ubuntu@dns
```

##### 동작 방식
---
- 암호화의 기본이 되는 대칭키, 비대칭키를 활용하여 Clinet-Server 연결
- 비대칭 키 방식
    - 공개 키(.pub) - 개인 키(.pem)로 한쌍을 이루며 서로의 정체 증명
    - E.g) Cloud server는 공개키를 발행하며, Client에게 개인 키를 해당 키를 기반으로 Cloud에 접속 요청
- 대칭 키 방식
    - 하나의 키를 만들어 서로에게 공유
    - 해당 키를 통해 당시에만 복호화하여 사용
    - E.g) 정보를 주고 받는 과정에서 Server가 키 값을 알려주고 암호화 하여 전달 → 사용자는 정보를 해독하기 위해서 키를 사용

<br>

##### 이슈 사항
---
AWS RDS → Redshift로 파이프라인 구성하여 사용하는 도중 SSH에러가 발생하여 Pipeline 중단 현상
- EC2 Ubuntu 내 openssh 버전을 올려서 세팅
- 최근 openssh 버전은 ssh-rsa 공개키 지원을 ssh-rsa → sha1으로 변경
- ssh 기본 설정 값 변경

```python
# server ssh log 확인 
$ tail -f /var/log/auth.log
>
userauth_pubkey: key type ssh-rsa not in PubkeyAcceptedAlgorithms [preauth]
Received disconnect from 222.106.177.9 port 49130:11:  [preauth]
Disconnected from authenticating user ubuntu 222.106.177.9 port 49130 [preauth]
Reloading OpenBSD Secure Shell server...
Received SIGHUP; restarting.
Reloaded OpenBSD Secure Shell server.
Server listening on 0.0.0.0 port 22.
Server listening on :: port 22.
```

- SSH 서버가 RSA Key type을 수락하지 않기 때문에 추가
- sshd_config 파일을 설정해주어야 한다. **`PubkeyAcceptedAlgorithms`** `+ssh-rsa`

```python
$ echo "PubkeyAcceptedAlgorithms +ssh-rsa" >> /etc/ssh/sshd_config
$ systemctl restart sshd
```

<br>

##### 참조
---
- [https://velog.io/@yjsworld/openssh-ssh-rsa-이슈](https://velog.io/@yjsworld/openssh-ssh-rsa-%EC%9D%B4%EC%8A%88)
- [https://sangchul.kr/entry/리눅스-userauthpubkey-key-type-ssh-rsa-not-in-PubkeyAcceptedAlgorithms](https://sangchul.kr/entry/%EB%A6%AC%EB%88%85%EC%8A%A4-userauthpubkey-key-type-ssh-rsa-not-in-PubkeyAcceptedAlgorithms)
- [https://docs.digitalocean.com/support/how-to-troubleshoot-ssh-connectivity-issues/](https://docs.digitalocean.com/support/how-to-troubleshoot-ssh-connectivity-issues/)