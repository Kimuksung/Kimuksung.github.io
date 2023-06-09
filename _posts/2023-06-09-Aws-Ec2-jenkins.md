---
layout: post
title:  "Jenkins 환경 구성"
author: Kimuksung
categories: [ Jenkins ]
tags: [ AWS, Jenkins ]
image: "https://i.ibb.co/xzTs9SF/jenkins.png"
comments: False
---


#### install docker

##### docker repository
---
- Install Library
    
    ```python
    $ sudo apt-get update
    $ sudo apt-get install \
        ca-certificates \
        curl \
        gnupg \
        lsb-release
    ```
    
- Docker GPG Key
    
    ```python
    $ sudo mkdir -p /etc/apt/keyrings
    $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    ```
    
- Repository setup
    
    ```python
    $ echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
      $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    ```
    

##### docker engine
---
- Install Docker Engine
    
    ```python
    $ sudo apt-get update
    $ sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
    ```
    
- check Docker instance
    
    ```python
    $ docker ps
    ```
    
    ```python
    # 에러 발생 시
    permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/containers/json": dial unix /var/run/docker.sock: connect: permission denied
    >
    sudo chmod 666 /var/run/docker.sock
    ```
    

##### Docker Run jenkins
---
- 일반적으로 Image로 바로 띄우는 용도
- 최근 Jenkins 이미지를 다운 받은 후
    
    ```python
    $ docker pull jenkins/jenkins:lts
    $ sudo docker run -d -p 8080:8080 -v /jenkins:/var/jenkins_home --name jenkins -u root jenkins/jenkins:lts
    ```
    
##### docker-compose 방식
---
```python
$ sudo apt install docker-compose
```

```python
#docker-compose.yml
version: "3"
services:
  jenkins:
    image: jenkins/jenkins:lts
    user: root
    volumes:
      - ./jenkins:/var/jenkins_home
    ports:
      - 8080:8080
```

```python
$ sudo docker-compose up -d
```

```python
$ mkdir jenkins

$ cd jenkins

$ echo 'version: "3"
services:
  jenkins:
    image: jenkins/jenkins:lts
    user: root
    volumes:
      - ./jenkins:/var/jenkins_home
    ports:
      - "9090:9090"' > docker-compose.yml
```

##### Docker-compose 추가 환경 구성
---
- Docker-compose 파일을 활용
- 요구 사항에 맞추어 Python과 Python Library 설치

```python
$ sudo apt install docker-compose
$ mkdir jenkins
$ cd jenkins
```

```python
# Dockerfile
FROM jenkins/jenkins:lts
USER root

RUN apt-get update && apt-get install -y python3.9 python3-pip

RUN pip3 install requests pandas openpyxl numpy sqlalchemy flask pymysql redshift-connector scikit-learn scipy seaborn matplotlib gspread bs4 oauth2client psycopg2 gspread-dataframe
```

```python
echo -e "FROM jenkins/jenkins:lts\nUSER root\n\nRUN apt-get update && apt-get install -y python3.9 python3-pip\n\nRUN pip3 install requests pandas openpyxl numpy sqlalchemy flask pymysql redshift-connector scikit-learn scipy seaborn matplotlib gspread bs4 oauth2client gspread-dataframe" > Dockerfile
```

```python
$ docker build -t "jenkins:python" ./
```

```python
$ docker run -p 8080:8080 -v host_directory:container_directory jenkins:python

$ sudo docker run -d -p 8080:8080 -v /jenkins:/var/jenkins_home --name jenkins -u root jenkins:python

```