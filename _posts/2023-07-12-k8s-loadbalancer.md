---
layout: post
title:  "LoadBalancer를 활용하여 배포"
author: Kimuksung
categories: [ K8S ]
tags: [ K8S ]
# 댓글 기능
comments: False
image: "https://i.ibb.co/c8SRf7t/k8s.png"
---

service의 기본 개념이 LoadBalancer 배포 방법을 알아보려고 합니다.

Node Port는 Port를 맵핑하여 배포하는 방식이기에, Pod가 죽었다 살아나고 Pod수가 증가하는 과정에서 Resource가 들게 됩니다.

이를 편리하게 하나의 IP로 Inbound를 설정하고 내부에서 골고루 분산처리하게 도와주는 LoadBalncer 기능을 알아보려고 합니다.

EKS와 같이 실 서비스에서는 바로 구성이 되는 것으로 알지만, 현재 Local Minikube환경에서는 이를 설정하기 위해 metallb를 설정해주어야 합니다. → 이는 다음 장에 구성할 예정입니다.

#### grafana 배포하기
---
- 간단하게 grafana를 배포하여 볼 예정입니다.
- Loadbalancer의 Targetport와 deployment의 port를 동일하게 맞춰주어야합니다.

```bash
$ chomd +x grafana_start.sh
$ ./grafana_start

# 터널링
$ minikube tunnel

$ kubectl get svc
```

```bash
# grafana_deploy.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana
  labels:
    app: grafana
spec:
  replicas: 1
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      containers:
        - name: grafana
          image: grafana/grafana:latest
          ports:
            - name: http
              containerPort: 3000
---
apiVersion: v1
kind: Service
metadata:
  name: grafana
  labels:
    app: grafana
spec:
  type: LoadBalancer
  ports:
    - name: http
      protocol: TCP
      port: 9999
      targetPort: 3000
  selector:
    app: grafana
```

```bash
# grafana_start.sh
kubectl apply -f grafana_deploy.yml
minikube service grafana
```

```bash
# grafana_delete.sh
kubectl delete -f grafana_deploy.yml
```

#### Web 페이지 배포하기
---
- workernode에 대한 정보를 노출하는 web 페이지를 구성하였습니다.
- LoadBalancer를 통하여 동일하게 접근하여도 Server-name이 다른 것을 볼 수 있습니다.
- 이 때, 터널링을 활용하게 되면 ExterIP가 localhost로 설정되어 접근이 가능합니다.

```bash
$ ./clusterip_start
$ minikube tunnel

$ kubectl get svc
```

```bash
# clusterip_start.sh
kubectl apply -f deployment.yml
```

```bash
# clustserip_delete.sh
kubectl delete -f deployment.yml
```

```bash
# deployment.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello
spec:
  replicas: 3
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
---
apiVersion: v1
kind: Service
metadata:
  name: hello
  labels:
    app: hello
spec:
  type: LoadBalancer
  ports:
    - name: http
      protocol: TCP
      port: 8080
      targetPort: 80
  selector:
    app: hello
```