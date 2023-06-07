---
layout: post
title:  "Access AWS EC2(Public-IP)"
author: Kimuksung
categories: [ EC2 ]
tags: [ AWS, EC2 ]
image: "https://ifh.cc/g/md8cYV.png"
comments: False
---

EC2 Instance 띄운 후 SSH Connection 연결 처리하였으나, Web으로 접근 시에 제대로 연결이 되었는지 알기 위하여 정리한 내용입니다.

아래는 진행한 과정입니다.

1. NACL, Security group이 제대로 설정되었는지 여부
2. Ping을 통하여 연결이 이루어지는지 확인
3. Instance 내에서 Curl을 통해 Response가 오는지 확인
4. 외부에서 Curl을 통해 Response가 오는지 확인
5. Instance에서 방화벽 해제 여부

<br>

##### 1. NACL, Security group
---
본인의 IP 공인 확인 = https://www.findip.kr/

- 공인 IP = **222.106.177.9**
- NACL과 Security Group에 SSH연결과 All Traffic 설정과 공인 IP 값에 해당되는 IP 대역을 설정합니다.

##### 2. Ping을 통하여 연결이 이루어지는지 확인
---
- 외부에서 Ping을 통해 실제로 Connection이 이루어지는지 봅니다.
- 이 때 TimeOut이 난다면 제대로 Connection이 이루어지지 않는 것입니다.

```python
$ ping ec2-public-ip
```

##### 3. Instance 내에서 Curl을 통해 Response
---
- Apache2를 이용한 Web Response 확인하기
- http://public-ip:80
- curl 명령어를 통해 Response가 온다면, 다음으로 진행하여 줍니다.
    
    ```python
    $ sudo apt-get update
    $ sudo apt install apache2
    $ systemctl status apache2
    $ sudo service apache2 start
    ```
    
    ```python
    $ curl localhost:80
    $ curl localhost:8080
    ```
    

##### 4. 외부에서 Curl을 통해 Response가 오는지 확인
---
- curl 명령어를 통해 Request에 대해 Response값이 오는지 확인하여 이상 없는지 체크합니다.

    ```python
    $ curl http://ec2-public-ip:80
    ```

##### 5. 방화벽 설정
---
- 참조 = https://webdir.tistory.com/206
- EC2 Instance가 Ubuntu로 설정하였기에 위 링크를 참조하여 Ubuntu 초기에 방화벽이 해제되어있지 않는 부분에 대해 변경하여 줍니다.
- 방화벽 설정
    
    ```python
    $ sudo ufw enable
    ```
    
- 방화벽 확인
    
    ```python
    $ sudo ufw status verbose
    ```
    
- 방화벽 추가
    
    ```python
    $ sudo ufw allow 80
    ```