---
layout: post
title:  "kubernetes configmap"
author: Kimuksung
categories: [ K8S ]
tags: [ K8S, configmap ]
# 댓글 기능
comments: False
image: "https://i.ibb.co/c8SRf7t/k8s.png"
---

#### configmap
---
- Config 값에 대해 미리 지정 후 Pod 배포 시 실행
- 아래 2가지의 yml 파일로 구성
    - configmap.yml
    - deployment.yml

mysql 배포를 통한 configmap 테스트를 하여본다.

<br>

#### 1. deployment 환경변수 직접 넣어 구성
---
- deployment에 env 환경변수를 직접 넣어 구성하기
- deployment.yml
    
    ```bash
    $ kubectl apply -f deployment.yml
    $ kubectl get pod -w
    
    $ kubectl exec -it deploy/mysql bash
    $ echo $MYSQL_ROOT_PASSWORD
    
    $ mysql -u root -p
    $ show databases
    
    $ kubectl delete -f deployment.yml
    ```
    
    ```bash
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
              value: uksungtest
            - name: MYSQL_DATABASE
              value: uksungtest
            ports:
            - name: http
              containerPort: 3306
              protocol: TCP
    ```
    

### 2. ConfigMap을 통해 ENV 설정

---

- configmap을 통하여 env 설정 분리
- deployment.yml에서 configmap의 env 참조
    
    ```bash
    $ kubectl apply -f configmap.yml
    $ kubectl apply -f deployment.yml
    $ kubectl get cm
    
    $ kubectl exec -it deploy/mysql bash
    $ echo $MYSQL_ROOT_PASSWORD
    $ echo $MYSQL_DATABASE
    
    $ mysql -u root -p
    $ show databases;
    
    $ kubectl delete -f deployment.yml
    $ kubectl delete -f configmap.yml
    ```
    
    ```bash
    # configmap.yml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: mysql-config
    data:
      MYSQL_ROOT_PASSWORD: uksungtest
      MYSQL_DATABASE: uksungtest
    ```
    
    ```bash
    # deployment.yml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: database1
    spec:
      selector:
        matchLabels:
          app: database1
      template:
        metadata:
          name: database1
          labels:
            app: database1
        spec:
          containers:
          - name: database1
            image: mariadb:10.7
            envFrom:
            - configMapRef:
                name: mysql-config
    ```
    
- 추가적으로 configmap이 중복되어 실행된다면 어떻게 될까?
    - configmap_duplicate.yml을 통해 동일한 name에 값을 추가한다.
    - 결과는 덮어씌워지기 때문에 duplicate 내의 password, database 설정
    - 즉, tt로 설정된다.
        
        ```bash
        # configmap_duplicate.yml
        apiVersion: v1
        kind: ConfigMap
        metadata:
          name: mysql-config
        data:
          MYSQL_ROOT_PASSWORD: tt
          MYSQL_DATABASE: tt
        ```
        

#### 3. configmap의 env-key만 가져오기
---
- configmap을 통하여 env 설정 분리
- deployment.yml에서 configmap의 key 참조
- **valuefrom**과 **configmapkeyref**를 통해 env의 key만 가져올 수 있다.
    
    ```bash
    $ kubectl apply -f configmap.yml
    $ kubectl apply -f deployment.yml
    $ kubectl get cm
    
    $ kubectl exec -it deploy/mysql bash
    $ echo $MYSQL_ROOT_PASSWORD
    $ echo $MYSQL_DATABASE
    
    $ mysql -u root -p
    $ show databases;
    
    $ kubectl delete -f deployment.yml
    $ kubectl delete -f configmap.yml
    ```
    
    ```bash
    # configmap.yml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: mysql-config
    data:
      MYSQL_ROOT_PASSWORD: uksungtest
      MYSQL_DATABASE: uksungtest
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
            env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                configMapKeyRef:
                  name: mysql-config
                  key: MYSQL_ROOT_PASSWORD
    ```
    
- 만약 여러개의 key를 가져오고 싶다면 어떨까?
    - key에다가 여러개를 입력하여 주면 에러가 발생한다.
    - 아래와 같이 key 하나당 valuefrom을 계속하여 지정해주어야 한다.
        
        ```bash
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
                    configMapKeyRef:
                      name: mysql-config
                      key: MYSQL_ROOT_PASSWORD
                - name: MYSQL_DATABASE
                  valueFrom:
                    configMapKeyRef:
                      name: mysql-config
                      key: MYSQL_DATABASE
        ```
        

#### volume + configmap을 활용한 mount
---
- configmap 정보를 → volume에 저장한다.
- volume 폴더를 마운트시킨다.
- mysql container는 config를 불러온다.
    
    ```bash
    # configmap.yml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: mysql-config
    data:
      MYSQL_ROOT_PASSWORD: uksungtest
      MYSQL_DATABASE: uksungtest
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
            volumeMounts:
            - mountPath: /tmp/config
              name: mysql-config
          volumes:
          - name: mysql-config
            configMap:
              name: mysql-config
    ```
    
- 그렇다면, 여러개의 config 환경을 반영하고 싶다면 어떻게 해야할까?
    - volumes에  각 config 파일들을 구성
    - volumemount에는 하나의 파일 경로만 가능하기 때문에 각각을 나누어서 mount 시켜줍니다.
    - 하나의 폴더 아래에 여러개로 나누어서 말이죠
    - 결과 → aws 값과 mysql-conf값 둘다 잘 나오는 것을 알 수 있습니다.
        
        ```bash
        # awsconf.yml
        apiVersion: v1
        kind: ConfigMap
        metadata:
          name: aws
        data:
          aws_access_key: aws_id
          aws_access_credential: aws_pw
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
                - configMapRef:
                    name: aws
                volumeMounts:
                - mountPath: /tmp/config
                  name: mysql-config
                  # must be unique
                - mountPath: /tmp/aws
                  name: aws-config
              volumes:
              - name: mysql-config
                configMap:
                  name: mysql-config
              - name: aws-config
                configMap:
                  name: aws
        ```
        
        ![](https://i.ibb.co/sKNH3nW/2023-07-14-2-12-27.png)
