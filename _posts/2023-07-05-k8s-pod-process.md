---
layout: post
title:  "Kubernetes pod 생성 과정"
author: Kimuksung
categories: [ K8S ]
tags: [ K8S ]
# 댓글 기능
comments: False
image: "https://i.ibb.co/c8SRf7t/k8s.png"
---

##### K8S의 기본
---
- `Namespace` = kube-system
- `MSA` 구조로 동작한다.
- 가장 기본이 되는 구조로써, default Namespace에 남아있지 않는다.
- Master = API Server + etcd(database) + Scheduler +
- Worker = Kubelet
    
    ```python
    $ kubectl get namespace
    $ kubectl get pod -n kube-system
    ```
    
    !https://i.ibb.co/5kRpX6x/2023-07-04-8-12-46.png
    
    !https://i.ibb.co/1fgD2H3/2023-07-04-8-08-03.png
    

기본 흐름 구조 부터 이해해보자

**`kubectl create`** 
- 명령으로 수행되는 과정을 나타내는 도식도
- `declarative 형태`로 API Server를 상태를 보고 동일한 상태로 만드려고 노력한다.

1) Kubectl → API Server Pod 생성 요청

2) APIServer는 etcd에 Node에 할당하지 않은 Pod 있다는 것을 Update

3-1) Scheduler는 etcd의 변경 사항을 API Server를 통해 watch

3-2) Pod를 실행할 Node를 선택

4) Kubelet은 생성할 pod를 watch해서 

5) API Server는 Kubelet을 통해 Pod가 노드에 잘 소속되어있는지 감시

6) Kubelet은 Docker에게 동작 및 결과 업데이트

7)  etcd에 update

![](https://i.ibb.co/yR791Yt/2023-07-04-8-08-22.png)