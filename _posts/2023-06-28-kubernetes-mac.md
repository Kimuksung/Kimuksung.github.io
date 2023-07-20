---
layout: post
title:  "MAC에서 Kubernetes 개발 환경 만들기"
author: Kimuksung
categories: [ K8S ]
tags: [ K8S ]
# 댓글 기능
comments: False
image: assets/images/k8s.png
---


일반적으로 많은 강의에서도 볼수 있듯이 K8S를 사용하기 위해서는 Virtual Box를 사용하여 구축한다.

MAC은 VirtualBox와 호환이 잘 되지 않고 지원도 하지 않기 때문에 다른 방안을 찾고 있었다.

- minikube
- kind

잘 공유되어있는 블로그를 발견하여 minikube로 진행해볼 예정이다.

<br>

##### K8S 설치하기
---
- 1.25.2 버전으로 설치되는데, m1 유저는 1.25.1 아래로 받아야 한다.

```python
$ brew install minikube
# 만약 문제가 발생한다면
$ minikube delete
$ minikube start --kubernetes-version=v1.23.1
```

```python
$ curl -Lo minikube https://github.com/kubernetes/minikube/releases/download/v1.25.1/minikube-darwin-arm64 \
  && chmod +x minikube
$ sudo install minikube /usr/local/bin/minikube
```

```python
$ minikube version
```

minikube에서는 가상 드라이버를 설정해야한다.  
M1에서는 **Docker만 지원**한다.

- Hyperkit
- Hyper-V
- Docker
- VirtualBox

```python
$ minikube start --driver=docker
```

![](https://i.ibb.co/60ftLx1/2023-06-28-8-08-30.png)

##### kubectl 설치
---
```python
$ brew install kubectl
```

##### Docker에 띄우기
---
- docker에서 local 환경에 아래와 같은 서비스를 실행

```python
$ docker-compose up -d
```

```python
docker-compose.yml
version: "3"

services:
  wordpress:
    image: wordpress:5.9.1-php8.1-apache
    environment:
      WORDPRESS_DB_HOST: mysql
      WORDPRESS_DB_NAME: wordpress
      WORDPRESS_DB_USER: root
      WORDPRESS_DB_PASSWORD: password
    ports:
      - "30000:80"

  mysql:
    image: mariadb:10.7
    environment:
      MYSQL_DATABASE: wordpress
      MYSQL_ROOT_PASSWORD: password
```

##### kubernetes 배포
---
- 아직 정확하게 돌아가는 동작 원리 및 구조는 파악하지 못하였다.
- 일단은 러프하게 다른 사람이 배포한 것을 보고 쫓아가면 어느정도 이해한 뒤 다시 접근할 예정

```python
# create cluster
$ kubectl apply -f wordpress-k8s.yml

# check cluster component
$ kubectl get all 
```

![](https://i.ibb.co/yYyvqSM/2023-06-28-8-18-58.png)

```python
# wordpress-k8s
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: mysql
  template:
    metadata:
      labels:
        app: wordpress
        tier: mysql
    spec:
      containers:
        - image: mariadb:10.7
          name: mysql
          env:
            - name: MYSQL_DATABASE
              value: wordpress
            - name: MYSQL_ROOT_PASSWORD
              value: password
          ports:
            - containerPort: 3306
              name: mysql

---
apiVersion: v1
kind: Service
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  ports:
    - port: 3306
  selector:
    app: wordpress
    tier: mysql

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: frontend
  template:
    metadata:
      labels:
        app: wordpress
        tier: frontend
    spec:
      containers:
        - image: wordpress:5.9.1-php8.1-apache
          name: wordpress
          env:
            - name: WORDPRESS_DB_HOST
              value: wordpress-mysql
            - name: WORDPRESS_DB_NAME
              value: wordpress
            - name: WORDPRESS_DB_USER
              value: root
            - name: WORDPRESS_DB_PASSWORD
              value: password
          ports:
            - containerPort: 80
              name: wordpress

---
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  type: NodePort
  ports:
    - port: 80
  selector:
    app: wordpress
    tier: frontend
```

##### k8s port-forwarding
---
- Mac에서는 각 서비스 마다 포트를 별도로 열어주어야 접속이 가능하다고 한다.
- 그렇지 않으면 포트가 열려있다고 하더라도 아래와 같이 연결이 되지 않는다.
    
    ![](https://i.ibb.co/b2J7S1j/2023-06-28-8-21-47.png)
    

```python
$ kubectl port-forward service/{서비스명} {로컬 포트}:{서비스 포트}
$ kubectl port-forward service/wordpress 4000:80
```

![](https://i.ibb.co/cxX6wtp/2023-06-28-8-22-52.png)

### K8S 서비스 및 아이피 확인하기

---

- ip - 외부와 연결된 ip 값
    - MAC이 아니였다면 해당 ip와 포트로 연결되어 보여진다고 함
        
        ![](https://i.ibb.co/Fgk6TTP/2023-06-28-8-41-30.png)
        
- [Service](https://minikube.sigs.k8s.io/docs/commands/service/) - Returns a URL to connect to a service

```python
$ minikube service {서비스명}
$ minikube ip
```

![](https://i.ibb.co/TYFfFq8/2023-06-28-8-41-43.png)

![](https://i.ibb.co/vYPmdpY/2023-06-28-8-41-21.png)

<br>

###### 참조
---
- [https://velog.io/@pinion7/macOs-m1-환경에서-kubernetes-시작하기](https://velog.io/@pinion7/macOs-m1-%ED%99%98%EA%B2%BD%EC%97%90%EC%84%9C-kubernetes-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0)