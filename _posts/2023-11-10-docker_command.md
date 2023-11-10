---
layout: post
title:  "Docker 기초"
author: Kimuksung
categories: [ Docker ]
tags: [ Docker ]
image: assets/images/docker.svg
comments: False
---

Docker를 사용하면서 사용했거나 필요하다고 생각되는 명령어 위주로 정리히보았습니다.

##### Container
- 공통 OS 환경에서 구성된 가상 컨테이너

##### Run
- -d : 백그라운드 옵션
- -p : 포트포워딩
- -v : 볼륨 마운트
- -rm : 내부 생성된 값을 제거(캐싱)
- command option : 실행 시킬 command를 뒤에 작성 ( CMD를 덮어쓰니 주의 )
- $PWD는 현재 경로 옵션

```bash
$ docker run -p 3306:3306 /usr/bin/hadoop:/var/bin/hadoop_home hadoop -d
$ docker run -p 3306:3306 -v $PWD/hadoop:/var/bin/hadoop_home hadoop -d

# command option
$ docker run --name test entry_test ps -aef
$ docker-compose -f target.yml up -d
```

##### Commit

```bash
docker commit container image
```

##### Stop

- Container 중지 사용 ( 즉시 중지 X )
- -t : 대기 시간 지정
- -f : 즉시 중지 옵션
- -v : mount volume 까지 삭제

```bash
docker stop container
docker stop -t second container
docker stop -f container
docker stop container1, container2, .. 
docker stop -v container
```

##### Kill

- 즉시 중지 시킨다. ( 치명적일 수 있으니 사용 X)

```bash
docker kill container
```

##### Access

- exec

```bash
# bash shell
docker exec -it container /bin/bash
```

##### Network

- link
    - 연결할 컨테이너 이름:alias
    - 현재는 network 옵션으로 사용하기에 deprecated
        
        ```bash
        docker run --name test -p 80:80 --link db:db nginx
        ```
        
- 컨테이너는 eth0이라는 네트워크 인터페이스를 할당 받고 Host는 다른 컨테이너와 연결을 위해 docker0라는 연결 통로를 만들어준다.
- network
    - bridge : 컨테이니끼리 통신하기 위함
    - host : host와 동일한 환경
    - None

```bash
docker network ls
docker network inspect bridge
```

<a href="https://postimg.cc/Yjm301SN" target="_blank"><img src="https://i.postimg.cc/Yjm301SN/docker-network.png" alt="docker-network"/></a><br/><br/>
<a href="https://postimg.cc/Xpfkc4dp" target="_blank"><img src="https://i.postimg.cc/Xpfkc4dp/ehto0.png" alt="ehto0"/></a><br/><br/>

##### 기타
- logs : 컨테이너 모니터링

```bash
docker logs container
# 실시간
docker logs container -f
```

- ps : 현재 구동중인 컨테이너

```bash
docker ps
# 전체 컨테이너
docker ps -a
```

- stats : resource 확인

```bash
docker stats
```