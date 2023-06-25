---
layout: post
title:  "Hadoop Rack Awareness Poclies"
author: Kimuksung
categories: [ Hadoop ]
#blog post image
image: "https://i.ibb.co/YyZgL6g/rack-placmement-policy-hdfs.png"
comments: False
---

Client가 Data를 저장할 때 여러 곳에 분산처리하여 저장한다.  
이 때 Rack이 장애가 날 수 있으니, 이를 방지하기 위함 + Failure Tolerance

##### Rack 인지 정책
---
목표 : RACK이 Terminate되어도 계속 유지되어야 함
1. 같은 Data Node에는 한개 이상의 동일한 Replica를 저장 불가
2. 같은 Rack 안에 2개 초과의 single block replica를 허용하지 않는다.
3. Rack 수 ≤ Replicas 수 이여야 한다. 

##### Data Block Replicas
---
- Default = 3 Replicas
- client로 부터 요청을 받은 DataNode에서는 첫번째 Replica를 저장한다.
- Second Replica는 같은 Rack의 다른 DataNode에 저장
- Third Replica는 다른 Rack에 있던 random DataNode에 저장


![t1](https://i.ibb.co/Ss5GdmD/image.gif)
![t2](https://i.ibb.co/YyZgL6g/rack-placmement-policy-hdfs.png)
![t3](https://i.ibb.co/kyt9Xmp/image.jpg)
