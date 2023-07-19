---
layout: post
title:  "kubernetes Secret"
author: Kimuksung
categories: [ K8S ]
tags: [ K8S, Secret ]
# 댓글 기능
comments: False
image: "https://i.ibb.co/c8SRf7t/k8s.png"
---

앞에서 경험한 **configmap**을 통하여 **환경변수를 동기화** 시켜 사용이 가능하다.

하지만, aws credentials값이나 git ssh 값은 대놓고 동기화 시켜 사용하는 것이 어렵다.

그렇게 해서 env 데이터 값을 감추어 사용하고자 하여 나온 개념이 secret이다.

물론 git을 통한 CI/CD를 구축하게 된다면, secret 값이 노출되기 때문에 이에 경우는 다른 방안을 찾아봐야한다.

<br>

#### secret이란?
---
- **password**, **key**와 같이 보안이 중요한 데이터를 container에 넣어 사용하는 경우
- configmap과 유사하게 사용
- 기본적으로 secret은 **etcd database**에 저장하며, base64 type으로 encrypt한다.

<br>

#### secretRef
---
- yml 파일 내에 envFrom을 선언하여 secret data를 가져온다.
- Secret kind라는 값 내에 stringData 에 key : value 형태로 원하는 정보를 저장하여 준다.
- conf.yml을 보면, MYSQL_ROOT_PASSWORD에 비밀번호를 설정한 것을 볼 수 있다.

```bash
$ chmod +x create_secret.sh
$ ./create_secret.sh
```

```bash
# create_secret.sh
kubectl apply -f conf.yml
kubectl apply -f deployment.yml
```

```bash
# conf.yml
apiVersion: v1
kind: Secret
metadata:
  name: uksung-secret
stringData:
  MYSQL_ROOT_PASSWORD: kimuksung
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-config
data:
  MYSQL_DATABASE: kubernetes
```

```bash
# deployment.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      name: mysql
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mariadb:10.7
        envFrom:
        - configMapRef:
            name: mysql-config
        - secretRef:
            name: uksung-secret
```

![](https://i.ibb.co/LNmCpNc/2023-07-19-9-45-12.png)

<br>

#### valueFrom
---
- 위에서 envFrom을 통하여 config 파일을 불러온것과 달리 env 아래에 valueFrom을 통하여 secretkey를 참조한다.
- secret object의 metadata name으로 매칭시켜 준 뒤, key 값을 넣어 반영한다.

```bash
# deployment.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      name: mysql
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mariadb:10.7
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: uksung-secret
              key: MYSQL_ROOT_PASSWORD
        - name: MYSQL_DATABASE
          valueFrom:
            configMapKeyRef:
              name: mysql-config
              key: MYSQL_DATABASE
```

<br>

#### volume을 활용
---
- envFrom으로부터 configmap, secret object를 가져온다. (name)
- env 값을 → volumes → name(key)에 맵핑하여 준다.
- volumeMount는 container에 동기화할 위치이며, 해당 공간에 name을 불러 데이터 sync를 하게 된다.

```bash
# deployment.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      name: mysql
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mariadb:10.7
        envFrom:
        - configMapRef:
            name: mysql-config
        - secretRef:
            name: uksung-secret
        volumeMounts:
        - mountPath: /tmp/config
          name: mysql-config
        - mountPath: /tmp/secret
          name: mysql-secret
      volumes:
      - name: mysql-config
        configMap:
          name: mysql-config
      - name: mysql-secret
        secret:
          secretName: uksung-secret
```

![](https://i.ibb.co/dtVWVK4/2023-07-19-10-27-38.png)