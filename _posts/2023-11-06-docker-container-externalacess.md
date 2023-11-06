---
layout: post
title:  "Docker Container 외부와 연결하기(네트워크)"
author: Kimuksung
categories: [ Docker ]
tags: [ Docker, Mysql ]
image: assets/images/docker.svg
comments: False
---

중간에 과제 혹은 코딩테스트를 진행하면서 글을 많이 쓰지 못하였다.  
과제를 거치며, 이 부분은 어떻게 하는지 궁금하여 글을 씁니다.  
Docker를 구성하는 과정에서 내부 컨테이너끼리는 통신하는 글을 엄청 많으나, 외부에서 서비스를 접속하는 글을 20페이지 정도 봐도 검색이 이상한지 찾지를 못하였다.  
그래서 궁금하여 진행하여 보았다. 대체 외부에서 접속하려면 어떻게 해야하는가? 
  
문제를 해결하는데, 꽤 긴 시간이 걸렸다..  
AWS VPC와는 완전 다른느낌이였으나 어쨋든 해결 완료  
  
문제를 정의하고 어떻게 접근하였는지를 기록

#### 문제 정의
---
Host 이의외 장비에서 Host의 Docker Container(Mysql)을 연결하려는데 컨테이너 끼리만 통신이 가능하다.  
  
  
##### 접근 방식
---
1. 외부 IP로 연결
- Mysql은 실제 연결되는지 구분하기가 어려워 Linux Container + Apache Server로 먼저 테스트
    
    ```python
    $ docker run -it --name mylinuxcontainer1 --rm ubuntu /bin/bash
    $ docker run -it --name mylinuxcontainer2 --rm ubuntu /bin/bash
    
    $ docker run -it --name mylinuxcontainer3 -d --net host --rm ubuntu /bin/bash
    $ docker run -it --name mylinuxcontainer4 -d --net host --rm ubuntu /bin/bash
    
    $ docker run -i -t --name webserver2 -p 80:80 ubuntu:14.04
    $ docker run -i -t -p 3306:3306 -p 175.113.78.186:7777:80 ubuntu:14.04
    $ docker run -i -t -p 80:80 -p 127.0.0.1:7777:80 ubuntu:14.04
    $ docker run -i -t -p 80:80 -p 0.0.0.0:7777:80 ubuntu:14.04
    
    $ apt-get update
    $ apt-get install -y net-tools iputils-ping
    $ apt-get install bridge-utils
    
    $ apt update
    $ apt-get install -y apache2
    $ service apache2 start
    
    $ ifconfig | grep inet
    ```
    

2. Docker Network 설정 Bridge → Host
- Bridge는 내부 통신을 위한 용도로 사용한다고 한다.
- Host는 Host의 네트워크와 동일하게 동작한다고 하지만, 내 문제는 Host까지도 접속이 되지 않았다라고 추측 (ping은 연결이 되었으나, port 연결이 되지 않았기에)
    ```python
    $ docker run -it --name mylinuxcontainer3 -d --net host --rm ubuntu /bin/bash
    $ docker run -it --name mylinuxcontainer4 -d --net host --rm ubuntu /bin/bash
    ```
    

3. Docker-compose bind-address 추가
- 127.0.0.1 / [localhost](http://localhost) / ip ..  추가
- Host를 거쳐서 통신할것으로 생각하였기에, 바인딩 주소를 별도로 만들어주어야하나? 라는 생각에 진행하여보았다.
    ```python
    command:
        bind-address=0.0.0.0
    ```
    

4. Port fowarding
- 일반적으로 도커의 포트포워딩을 하게 되면 접속이 가능하다. 나도 그런 줄 알았다. 매번 Host에서 확인하였기에
- 하지만, 이방법으로도 외부 연결은 되지 않아 조금더 앞단계에서 연결을 시도하여본다.
    ```python
    ports:
    	- 3306:3306 # HOST:CONTAINER
    ```
    

5. 방화벽/포트 해제
- MAC OS를 사용하다보니, 방화벽을 별도로 열어주지 않아 통신이 되지 않나?라는 생각에서 변경하여 보았다.    
    ```python
    $ sudo pfctl -d
    $ sudo vi /etc/pf.conf
    # rdr pass inet proto tcp from any to any port 3306 -> 127.0.0.1 port 3306
    # rdr pass inet proto tcp from any to any port 3306
    # pass in proto tcp from any to any port 8080
    ```
    

6. 사용중이 포트 죽이기
- 혹시나, 해당 포트를 먼저 선점하고 있는 프로세스가 있을까봐 진행하였다.
- 생각해보면 해당 포트를 누가 사용하고 있다면, Container가 실행을 어떻게 했지?    
    ```python
    $ sudo lsof -PiTCP -sTCP:LISTEN
    $ sudo lsof -i :8080
    $ udo kill -9 {PID}
    $ netstat -an | grep LISTEN
    ```
    
7. Ping과 Telnet 연결 확인
- ping : IP response O
- telnet : IP, Port response X
- 공인 아이피로 접속하는게 맞는가?
    
    ```python
    $ ping ip
    $ telnet ip port
    ```


8. 컴퓨터 Local IP로 연결
    - 지금까지 내 Public ip로 진행하였는데, 혹시나 Local IP로 접근해야 가능한지 궁금해서 테스트
    - 공유기 라우팅 관련 글을 보고 실제 IP가 다른가?
    - 회사망(공용 사무망)에서 연결 테스트 시 접근 가능
    - But, 외부에서 접근 시 접근 불가능
    
    ```python
    $ ifconfig | grep inet
    ```

    <a href="https://ibb.co/bQ7byBY"><img src="https://i.ibb.co/YPb0KkJ/2023-11-06-9-47-12.png" alt="2023-11-06-9-47-12" border="0"></a><br /><a target='_blank' href='https://nonprofitlight.com/ny/new-york/garden-of-eden-foundation-inc'></a><br />

9. 라우팅 설정을 하여 준다.
- 집에서는 다른 Host는 물론 같은 와이파이망에서도 접근이 불가능하다..
- 와이파이에도 포트포워딩이 있을것이고, 라우팅이 문제일 수 있다라는 네트워크 때 배운 내용이 생각이 나서 적용해보았다.
- KT를 사용중에 있어, http://homehub.olleh.com:8899/nat/portfwd 로 접근
- 초기 id = ktuser / pw = homehub
- 포트포워딩한 mysql 포트와 Host의 ifconfig | grep inet 값을 넣어주면 된다.
- 집의 다른 PC와 외부 지인의 컴퓨터에서 접속 요청 시 테스트 성공

    <p class="mb-5"><img class="shadow-lg" src="{{site.baseurl}}/assets/images/kt_router.png" /></p>

##### 이해한 내용(결론)

- 같은 와이파이 환경과 NAT를 사용하여 구성된다면, Local IP로 접근이 가능하다.
- 외부 네트워크 환경에서는 라우터가 접근을 막고 있기 때문에 포트포워딩 설정과 방화벽 설정을 풀어서 접근해야 한다.

- 동일한 와이파이 환경
    - Local IP - O
    - Public IP - X
- 외부 네트워크 환경
    - Local IP - X
    - Public IP - O