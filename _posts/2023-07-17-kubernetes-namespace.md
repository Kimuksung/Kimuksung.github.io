---
layout: post
title:  "kubernetes Namespace"
author: Kimuksung
categories: [ K8S ]
tags: [ K8S, Namespace ]
# 댓글 기능
comments: False
image: assets/images/k8s.png
---

#### Namespace
---
- 컨테이너를 논리적인 영역으로 나누는 역할
- Database의 Schema와 같은 개념
- CPU, Resource 할당을 제한 할 수 있다.

일반적으로 team단위로 사용하는 것으로 알고 있다.

Data platform, Data analysist .. 

<br>

#### api resource
---
Namespace 범위 Resource

- Namespace에 dependent한 resource
- Pod, Deployment, Service, Ingress, Secret, ConfigMap, Role ..

```bash
$ kubectl api-resources --namespaced=true
```

Cluster 범위 API Resource

- Namespace와 다르게 모든 Namespace에서 같이 사용된다.

```bash
$ kubectl api-resources --namespaced=false
```

#### 기본 Namespace
---
- Default
    - 기본적으로 할당되는 Namespace로 별도의 옵션을 추가하여 주지 않는다면 default
- kube-system
    - System에 의해 생성되는 Object를 관리하기 위한 Namespace
    - 관리자 Namespace
- kube-public
    - 모든 사용자로부터 접근 가능
- kube-node-lease
    - Cluster 내 노드 연결 정보를 관리하기 위한  Namespace
    

간단하게 test1, test2라는 Namespace nginx를 띄어 확인하여 본다.

```bash
#namespace_test.sh
kubectl create namespace test1
kubectl create namespace test2
kubectl apply -f . -n test1
kubectl apply -f . -n test2

minikube service hello -n test1
minikube service hello -n test2
```

```bash
kubectl describe service -n test1
```

```bash
# deployment.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      name: hello
      labels:
        app: hello
    spec:
      containers:
      - name: nginx
        image: nginxdemos/hello:plain-text
        ports:
        - name: http
          containerPort: 80
          protocol: TCP
```

```bash
# service.yml
apiVersion: v1
kind: Service
metadata:
  name: hello
  labels:
    app: hello
spec:
  type: NodePort
  ports:
  - name: http
    protocol: TCP
    port: 8080
    targetPort: 80
  selector:
    app: hello
```

![](https://i.ibb.co/3sBMKnN/2023-07-17-7-54-50.png)

![](https://i.ibb.co/QHnRkpZ/2023-07-17-7-55-21.png)

![](https://i.ibb.co/WWkhDgj/2023-07-17-7-55-52.png)

![](https://i.ibb.co/pK9dkqh/2023-07-17-7-56-05.png)

#### Namespace resource 제한하기
---
- LimitRange
    - Pod, Container를 만드는 경우 자원 기본 할당량 설정
    - 최대, 최소값을 두어 동작하게 한다.
- resourcequota
    - 전체적인 resource를 제한
    - Cpu, Memory, Volume의 총합을 제외
    - 생성 가능한 resource(pod,service,deployment) 개수 제한이 가능

```bash
kubectl apply -f namespace.yml
kubectl apply -f resource_quota.yml
kubectl apply -f limit_range.yml
kubectl apply -f pod.yml
kubectl apply -f unpod.yml
```

```bash
#namespace.yml
apiVersion: v1
kind: Namespace
metadata:
  name: team1
```

```bash
# limit_range.yml
apiVersion: v1
kind: LimitRange
metadata:
  name: limit-range
  namespace: team1
spec:
  limits:
  - type: Container
    default:
      memory: 128Mi
      cpu: 100m
    defaultRequest:
      memory: 64Mi
      cpu: 50m
    max:
      memory: 1Gi
      cpu: 1000m
    min:
      memory: 16Mi
      cpu: 10m
  - type: Pod
  - type: PersistentVolumeClaim
    max:
      storage: 1Gi
    min:
      storage: 100Mi
```

```bash
# resource_quota.yml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: object-count-quota
  namespace: team1
spec:
  hard:
    limits.cpu: "5000m"
    limits.memory: "8Gi"
    count/pods: 10
    count/replicationcontrollers: 10
    count/replicasets.apps: 10
    count/deployments.apps: 10
    count/statefulsets.apps: 10
    count/jobs.batch: 3
    count/cronjobs.batch: 3
    count/services: 5
    count/services.nodeports: 0
    count/services.loadbalancers: 0
    count/configmaps: 10
    count/secrets: 10
    count/persistentvolumeclaims: 5
    count/resourcequotas: 3
```

```bash
# pod.yml
apiVersion: v1
kind: Pod
metadata:
  name: test
  namespace: team1
spec:
  containers:
  - name: nginx
    image: nginxdemos/hello:plain-text
    ports:
    - name: http
      containerPort: 80
      protocol: TCP
```

```bash
# unpod.yml
apiVersion: v1
kind: Pod
metadata:
  name: unavailable
  namespace: team1
spec:
  containers:
  - name: nginx
    image: nginxdemos/hello:plain-text
    ports:
    - name: http
      containerPort: 80
      protocol: TCP
    resources:
      limits:
        cpu: 1
        memory: 512Mi
```