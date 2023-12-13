---
layout: post
title:  "Airflow Kubernetes Exector로 실행하기"
author: Kimuksung
categories: [ Airflow, Kubernetes ]
tags: [ Airflow, Kubernetes ]
image: assets/images/airflow.png
comments: False
featured: True
---

Airflow를 다른 Executor를 활용해서 구성해보고 실행해보려고 합니다.

이번 정리에서는 상세한 내용 파악까지는 못하여 구성한 코드와 어떻게 실행시켰는지를 적어두려합니다.

들어가기 전에 Docker와 Kunbernetes는 무슨 차이길래 Kubernetes를 지속적으로 사용하는 것일까요?

아래는 제 의견입니다. (틀릴수도..)

- 둘 다, Container 기반으로 동작시킨다는 점에서는 같습니다.
- 하지만, Docker는 가상 환경에 원하는 Computing Resource를 설정하여 Container 구성
- Kubernetes는 **`가용성`**을 위하여 가상 환경 Container가 지속적으로 띄워져있어야합니다. 그렇기에 계속하여 원하는 상태를 체크하고 동일하게 구성합니다.
- 결론 : Resource를 원할 때 마다 구성 할 수 있으며, 가용성을 지속적으로 유지하기 위함

<br>

##### Kubernetes Exector
---
- Kubernetes in Docker
- Master(Control components) + Worker(webserver, schedulers, mysql)을 구성하여 실행시켜보겠습니다.

- 아래 Airflow+Kubernetes를 가져와 사용하겠습니다.

```bash
$ git clone https://github.com/maxcotec/airflow-on-kubernets.git
$ ./create_cluster_with_registry.sh
```

<img src="https://kimuksung.github.io/assets/images/kubernetesexecutor1.png"/>
<br/>

- Airflow를 구성할 Dockerfile 설정 및 빌드를 진행합니다.
- build 시 containerid와 tag 값은 원하는 대로 수정해서 사용하여도 됩니다.
- 그럼 아래 이미지에서 볼 수 있듯이 이미지로 구성된 것을 볼 수 있습니다.

```bash
$ cd ../airflow-dags
$ docker build . -t airflow_first_dag:0.1.0

# dockerfile 수정
# https://stackoverflow.com/questions/76585758/mysqlclient-cannot-install-via-pip-cannot-find-pkg-config-name-in-ubuntu
# apt 추가 시 spacebar 주의
FROM apache/airflow:2.3.3-python3.8 as build

LABEL maintainer="Maxcotec <maxcotec.com>"
LABEL com.maxcotec.docker=true
LABEL com.maxcotec.docker.module="airflow"
LABEL com.maxcotec.docker.component="airflow"
LABEL com.maxcotec.docker.airflow.version="2.3.3"

USER root
RUN apt-get autoremove -yqq --purge && apt-get clean && \
    apt-get update && apt-get -y upgrade && \
    apt-get -y install default-libmysqlclient-dev && \
    apt-get install -y gcc && \
    apt-get install -y pkg-config && \
    sudo pip install mysqlclient

USER airflow

COPY --chown=airflow:root dags /opt/airflow/dags
```

<img src="https://kimuksung.github.io/assets/images/kubernetesexecutor2.png"/>
<br/>

- 실행하기 앞서, k8s는 주로 command로 설정해야하는데 이를 도와주는 k9s가 있어 사용해보았습니다.
- ui가 빈약하긴 하지만 직관적이고 초보자라 그런지 사용하기 괜찮았습니다.
- command는 `:` 를 활용하여 검색이 가능합니다.

```bash
$ brew install k9s
$ k9s
# 동작 방법
":" = command
"shift+f"= port-forwarding
":namespace"
":pod"
":Portforwarding"
```

- 현재 kubernetes의 config 설정값을 맞춰줍니다.

```bash
$ kubectl config get-contexts
$ kubectl config use-context kind-kind
```

- 여러 설명들이 있었지만, 아래에서 image를 잘못부르면 에러가 발생하며 실행되지 않으니 주의하셔서 설정해야합니다.
- 위 docker images의 `repository`와 `tag` 값을 넣어서 구성하여 줍니다.

```bash
$ helm upgrade --install airflow . --namespace default --values values.yaml

# values.yml
# docker images로 조회하여 나오는 repository/tag로 구성
airflow:
  dags_image:
    repository: localhost:5001/airflow_first_dag    # repository URL
    tag: 0.1.0                                      # airflow dags image version
    pull_policy: Always                             # change this to `Always` if changes applied on same tagged image.
  webserver:
    username: admin
    password: "12345678"
    secret_key: topsecret
  configs:
    fernet_key: 9jgWYeShQAYRS5E4Gu_n9GoGAuml4n50IpS-jifVqdA=

mysql:
  host: mysql
  username: airflow
  database: airflow
  password: "12345678"
```

<img src="https://kimuksung.github.io/assets/images/kubernetesexecutor3.png"/>
<br/>

<img src="https://kimuksung.github.io/assets/images/kubernetesexecutor4.png"/>
<br/>

<img src="https://kimuksung.github.io/assets/images/kubernetesexecutor5.png"/>
<br/>

<br>

##### Airflow 접근
---
- http://localhost:8080/home
- 위 helm을 활용해서 띄워진 pod의 webserver를 port-forwarding하여 접속이 가능하도록 설정하여 줍니다.
    
    ```bash
    # 첫글자 대문자로 해야합니다.
    $ :PortFowards
    ```

    <img src="https://kimuksung.github.io/assets/images/kubernetesexecutor6.png"/>
    <br/>
    
    <img src="https://kimuksung.github.io/assets/images/kubernetesexecutor7.png"/>
    <br/>

    <img src="https://kimuksung.github.io/assets/images/kubernetesexecutor8.png"/>
    <br/>

    <img src="https://kimuksung.github.io/assets/images/kubernetesexecutor9.png"/>
    <br/>
    
<br>

##### 참조
---
- https://www.youtube.com/watch?v=RqSYh3UI_Is
- https://github.com/maxcotec/airflow-on-kubernets/tree/hello-world-dag
- https://engineering.linecorp.com/ko/blog/data-engineering-with-airflow-k8s-1
- https://engineering.linecorp.com/ko/blog/data-engineering-with-airflow-k8s-2