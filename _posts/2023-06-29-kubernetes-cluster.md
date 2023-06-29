---
layout: post
title:  "Kubernetes cluster"
author: Kimuksung
categories: [ K8S ]
tags: [ K8S ]
# 댓글 기능
comments: False
image: "https://i.ibb.co/c8SRf7t/k8s.png"
---

##### Kubernetes Cluster란?
---
- K8S를 배포하면, K8S Cluster를 실행하고 있다고 생각하면 된다.
- Master Node + Worker Node로 구성

*앞서 구성한 minikube는 하나의 Node = Master + Worker 둘다 사용 

![](https://i.ibb.co/ZcTNhgr/k8s-master-slave.png)

<br>

##### Master Node
---
- Control Plane과 같은 개념
- 1~n개 까지 홀수개의 Node로 구성
- Cluster 상태 관리 + Command 처리
- Client(Schduler+controller-manager) - Server(Api server) - Data(etcd)의 역할을 분배

Scheduler  
- Api server와 통신하는 Component
- Worker Node Resource(cpu,memory, .. ) 상태 관리
- 새로 생성된 pod detect
- resource가 남는 worker Node를 선택하여 pod 배포하는 역할

Controller-manager
- Controller process 관리
- Kube Controller Manager, Cloud Control Manager(aws,gcp)
- api resource(pod, deployment, service, .. )을 관리하는 Controller 배정
- Api 서버 health check
- reconcile = etcd resource 상태 비교 → 동기화

kube api server
- API 제공

etcd
- key-value 저장소
- Cluster 상태 저장

<br>

##### Worker Node
---
- 실질적으로 실행되는 Node
- Container Runtime 위에서 Pod가 실행되는게 기본 형태
- System Component(kubelet, kube-proxy, network-addOn) 등의 컴포넌트 구성

kubelet
- Node 별 구성하고 있는 Component
- Master Node의 API server와 통신하며, Resource 상태 보고
- Worker Node의 Container Runtime과 통신
- 노드 내의 컨테이너 LifeCycle 관리

CRI
- Container Runtime Interface
- kubelet과 다양한 Container runtime고 통신
- Docker와 같이 Container를 관리해주는 역할

kube-proxy
- Node 내부적으로 proxy, Load Balancer 역할 수행

<br>

##### WorkLoad
---
- Pod
    - Container에서 동작하기 위한 Application 최소 단위
- pod의 집합으로, 실질적으로 동작하는 App resource들의 모음
- **App이 동작하기 위한 Pod들이 올바르게 실행될 수 있음을 보장**