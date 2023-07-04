---
layout: post
title:  "EC2(Jenkins) - Redshift"
author: Kimuksung
categories: [ EC2, Jenkins, Redshift ]
tags: [ AWS, EC2, Jenkins, Redshift ]
image: "https://i.ibb.co/xzTs9SF/jenkins.png"
comments: False
---

#### Jenkins-Redshift에서 연결
---
결과 - Private Ip Inbound 세팅

가설1) Public Ip Inbound 세팅

- Inbound를 설정하여주었음에도(Public Ip) Connection timeout이 발생하는 문제 발생

```python
$ curl ifconfig.me
```

가설2) Private Ip Inbound 세팅

- 같은 VPC내에 있는 Public Subnet(Jenkins)과 Private Subnet(Redshift)
- EC2 Private IP로 Inbound 설정하여 주면 연결이 된다.