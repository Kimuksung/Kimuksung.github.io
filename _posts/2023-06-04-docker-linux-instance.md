---
layout: post
title:  "Install multiple Linux Instance "
author: Kimuksung
categories: [ Linux ]
tags: [ Linux, Docker ]
image: "https://ifh.cc/g/1dT5w9.jpg"
comments: False
---

##### Dockerfile 설정
---
- Docker가 미리 설치되어있다는 가정하에 진행합니다.
- 여러 테스트 환경을 구축할 때 사용합니다.
- sudo command 까지 설치하여 줍니다,.

```python
# linux-file
FROM ubuntu:latest

# Update the repository and install basic packages
RUN apt-get update -y
RUN apt-get upgrade -y
RUN apt-get install -y git
RUN apt install sudo
```

##### Docker-compose.yml 설정
---
- context : 빌드할 파일 위치
- dockerfile : 빌드할 파일 명

```python
# docker-compose.yml
version: "3"
services:
  server1:
    build: 
      context: .
      dockerfile: linux-file
    ports:
      - "8083:8083"
    stdin_open: true
    tty: true

  server2:
    build: 
      context: .
      dockerfile: linux-file
    ports:
      - "8084:8084"
    stdin_open: true
    tty: true
```

- 빌드 시 사용합니다.

```python
$ docker-compose -f docker-compose.yml build
$ docker-compose --build
```
