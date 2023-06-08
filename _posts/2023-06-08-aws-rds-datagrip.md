---
layout: post
title:  "Access AWS RDS(Mysql) - Datagrip"
author: Kimuksung
categories: [ Datagrip, AWS ]
tags: [ AWS, Datagrip ]
image: "https://i.ibb.co/hMxpK05/datagrip.jpg"
comments: False
---

Datagrip을 통하여 여러 Database 환경을 빠르고 쉽게 접근하기 위한 세팅 방법입니다.

##### DataGrip 설치
---
```bash
$ brew install datagrip
```

##### Database 연동
---
상황에 Database Driver에 맞추어 연결해야합니다.
저는 AWS RDS Mysql을 연결하기 때문에 아래와 같이 설정하였습니다.
- Driver = Amazon Aurora Mysql
- aws mysql cluster public link
- port : 3306
- user, password : Database id,pw
- database : database-name
![test](https://i.ibb.co/qsLSvHV/2023-06-08-1-40-07.png)
    
<br>

##### ssh connection 세팅
---
- aws ssh url
- image    
![test](https://i.ibb.co/xg6pKhh/2023-06-08-6-45-44.png)
    
    

- ssh pem으로 연결
- host : aws instance public ip
- username : username
- authenitcation type : Key Pair
- Private key file : pem 경로 및 pem 파일
![test](https://i.ibb.co/yPD1N7G/2023-06-08-6-47-59.png)
