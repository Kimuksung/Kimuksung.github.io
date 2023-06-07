---
layout: post
title:  "Linux Install Docker "
author: Kimuksung
categories: [ Linux ]
tags: [ Linux, Docker ]
image: "https://ifh.cc/g/1dT5w9.jpg"
comments: False
---

##### uninstall
---
```python
$ for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

##### install apt repository
---
- 도커를 설치에 필요한 repository를 설치하여 줍니다.

```python
$ sudo apt-get update
$ sudo apt-get install ca-certificates curl gnupg
```

```python
$ sudo install -m 0755 -d /etc/apt/keyrings
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
$ sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

```python
$ echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

##### install docker engine
---
- 업데이트 후 도커 파일을 업데이트하여 줍니다.

```python
$ sudo apt-get update
```

```python
$ sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

##### check docker
---
```python
$ docker ps
```

##### 참조
---
https://docs.docker.com/engine/install/ubuntu/