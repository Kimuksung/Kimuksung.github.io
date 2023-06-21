---
layout: post
title:  "Spark-Aws RDS SSH로 연결하기"
author: Kimuksung
categories: [ Spark, AWS, RDS ]
#blog post image
image: assets/images/emr_spark.png
comments: False
---

ssh를 통한 연결을 해보려고 하는데, 몇일 째 시도중이지만 개선되지 않아 지금까지 햇던 것들을 정리하려고 한다.

1. SparkSession을 활용한 Direct 방법
-> 될리가 없다.
2. SSHTunnel을 활용한 접근 방식
- 터널링을 열어주고 아래와 같이 실행시킨다.
- 임의로 13306이라는 포트로 연결
- N옵션을 주어 Local에서도 실행되게끔
```ssh -i {pem 경로} -N -L 13306:rds-instance:rds-port ubuntu@instance```

- 터널링에서 13306 -> 3306 으로 포트포워딩된 상태에서 로컬에서 mysql로 접속
```mysql -h 127.0.0.1 -P 13306 -u userid -p```
위와 같이 하면 command shell에서는 동작한다.

다만 해주고 싶은 부분은 pyspark를 통해 바로 건드리고 싶은것이다..
SSHTunnelForwarder과 jdbc 연결을 통해 접근하여 보았으나, ec2를 거치지 않고 운영되는 rds만 접속이 될 뿐 ec2를 거치는 rds는 아직 부족하다.
```
from sshtunnel import SSHTunnelForwarder
from pyspark.sql import SparkSession
import os

pem_file_dir = os.path.dirname('pem/')
print("Pem File Directory:", pem_file_dir)
# pem 파일 리스트 출력
pem_files = os.listdir(pem_file_dir)
print("Pem Files:")
for file in pem_files:
    print(file)
    
# SSH 설정
pem_file_path = ""  # pem 파일 경로
ssh_username = ""  # SSH 사용자명
ssh_hostname = ""  # SSH 서버 주소


# RDS 설정
rds_hostname = ""  # RDS의 호스트명
rds_username = ""  # RDS 사용자명
rds_password = ""  # RDS 패스워드
rds_database = ""  # RDS 데이터베이스명
rds_table = ""

try:
    with SSHTunnelForwarder(
        (ssh_hostname, 22),
        ssh_username=ssh_username,
        ssh_pkey=pem_file_path,
        remote_bind_address=(rds_hostname, 3306),
        local_bind_address=('localhost', 3306)
    ) as tunnel:
        print('--start ssh--')
        tunnel.start()
        print('bind port : ' , tunnel.local_bind_port)
        bing_port = tunnel.local_bind_port

        # MySQL에 연결
        jdbc_url = f"jdbc:mysql://localhost:{bing_port},{rds_hostname}:3306/{rds_database}?enabledTLSProtocols=TLSv1.2"
        print(jdbc_url)
        df = spark.read \
            .format("jdbc") \
            .option("url", jdbc_url) \
            .option("driver", "com.mysql.cj.jdbc.Driver") \
            .option("dbtable", rds_table) \
            .option("user", rds_username) \
            .option("password", rds_password) \
            .load()

        df.show(5)
        df.printSchema()
except BaseException as e:
    print('Problem is --> ', e)
finally:
    if tunnel:
        tunnel.close()

```