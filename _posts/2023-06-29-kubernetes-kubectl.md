---
layout: post
title:  "Kubectl"
author: Kimuksung
categories: [ K8S ]
tags: [ K8S ]
# 댓글 기능
comments: False
image: "https://i.ibb.co/c8SRf7t/k8s.png"
---

##### kubectl
---
- kubectl 명령어를 통하여 kubernetes control를 한다.
- kuberenetes의 상태를 확인하고 원하는 상태를 요청

##### 1. apply
---
- yml 상태를 작성 후 적용

```python
$ kubectl apply -f {filename.yml}
```

##### 2. get
---
- type
    - pod
    - service
    - deploy
- all
    - 전체 조회
- -o
    - 출력 타입
    - json
    - yaml
    - wide

```python
$ kubectl get {type},{type} ..
$ kubectl get all
$ kubectl get {type} -show-labels
$ kubectl get {type} -o json
```

##### 3. describe
---
- resource의 상세한 상태 확인
- get을 통하여 원하는 resource의 name을 알아야 조회 가능

```python
$ kubectl describe {type}/{resource}
```

##### 4. delete
---
- 특정 resource 삭제
- get을 통하여 삭제를 원하는 resource의 name을 알아야 가능
- f 옵션
    - 파일과 관련된 리소스 전체 제거
- delete option을 통해 삭제하여도 Replicaset이 있다면 유지되어있다.

```python
$ kubectl delete {type}/resource
$ kubectl delete -f {resource-filename}
```

##### 5. logs
---
- pod의 log 조회
- -c 옵션
    - 하나의 pod에 여러 container가 존재한다면, -c 옵션으로 container 지정해서 로그 보는 것이 가능하다.
- -f
    - 실시간 로그 옵션

```python
$ kubectl logs {pod-name}
$ kubectl -f {pod-name}
$ kubectl logs {pod-name} -c {container-name}
```

##### 6. exec
---
- docker container 접속과 유사
- pod name을 알아야 접속 가능
- container 접속 시 주로 사용
- -it
    - 컨테이너 상태를 확인하는 경우
- -c
    - container 를 지정

```python
$ kubectl exec {pod-name} --it /bin/bash
$ kubectl exec {pod-name} --f {container-name}
```

###### 참조
---
- [https://velog.io/@pinion7/macOs-m1-환경에서-kubernetes-시작하기](https://velog.io/@pinion7/macOs-m1-%ED%99%98%EA%B2%BD%EC%97%90%EC%84%9C-kubernetes-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0)