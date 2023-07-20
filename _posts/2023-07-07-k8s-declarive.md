---
layout: post
title:  "Kubernetes Imperative vs Declarative"
author: Kimuksung
categories: [ K8S ]
tags: [ K8S ]
# 댓글 기능
comments: False
image: assets/images/k8s.png
---

Kubernetes로 Grafana 구성을 해본다.

#### Imperative
---
- kubectl을 통하여 일일히 액션

```bash
$ kubectl create deployment grafana --image=grafana/grafana --port=3000
$ kubectl get pod -w

$ kubectl expose deployment grafana --type=NodePort --port=80 --target-port=3000
$ kubectl get services -w

$ minikube service grafana
```

```bash
$ kubectl delete service/grafana
$ kubectl delete deployment/grafana

```

#### Declarative
---
- K8S가 지향하는 바로 Spec을 보고 다른 Kubelet이 보고 반영
- Docker-compose.yml과 같이 선언하여 사용

```bash
$ chmod +x grafana_start.sh
$ chmod +x grafana_delete.sh

$ ./grafana_start.sh
$ ./grafana_delete.sh
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
  type: NodePort
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 3000
  selector:
    app: grafana
```