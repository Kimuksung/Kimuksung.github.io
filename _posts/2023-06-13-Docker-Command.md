---
layout: post
title:  "Docker Command"
author: Kimuksung
categories: [ Docker ]
tags: [ Docker ]
image: "https://i.ibb.co/6nKVdtG/docker.png"
comments: False
---

##### docker ps vs docker ps -a
---
- `docker ps` = 현재 실행 중인 컨테이너 목록
- `docker ps -a` = 현재 실행 중인 컨테이너 + 멈춘 컨테이너

```bash
$ docker ps
$ docker ps -a
```

##### docker kill vs docker rm
---
- `docker kill`
    - 실행 중인 Docker Container를 강제 종료
    - 안전하게 종료되지 않는다.
    - SIGKILL syscall을 통해 컨테이너를 종료 → 내부 프로세스 정상적으로 종료되지 않을 수 있다.
- `docker rm`
    - Docker Container를 삭제
    - 컨테이너가 실행 중인 상태에서는 삭제 불가
    - docker kill → docker rm / docker rm -f
- `docker stop`
    - 실행 중인 Docker Container를 안전하게 종료
    
    ```bash
    $ docker kill container
    $ docker rm container
    $ docker rm -f container
    $ dpcler stop container
    ```
    

##### docker compose
---
- `docker-compose up`
    - Docker compose에 정의된 모든 서비스 컨테이너 한번에 생성 및 실행
- `docker-compose down`
    - docker compose에 정의된 모든 서비스 컨테이너 정지 및 삭제
- `docker-compose -f`
    - docker compose file name 지정하는 옵션
- `docker-compose start`
    - docker compose에 특정 서비스 컨테이너를 실행
- `docker-compose stop`
    - docker compose에 특정 서비스 컨테이너를 정지

```bash
$ docker-compose up 
$ docker-compose down
$ docker-compose -f file_name.yml up -d
$ docker-compose start jenkins
$ docker-compose stop jenkins
$ docker-compose exec -it continaer /bin/bash
```