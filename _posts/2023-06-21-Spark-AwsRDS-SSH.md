---
layout: post
title:  "Spark로 Private AWS RDS 연동"
author: Kimuksung
categories: [ Spark, AWS, RDS ]
#blog post image
image: assets/images/emr_spark.png
comments: False
---

Python의 도움을 받아 RDS를 연결하여 Spark를 활용한 전처리를 시도 중.  
Node -> EC2 -> AWS RDS Mysql에 있는 데이터를 가져오는 것이 목표.  
ssh를 통한 연결까지는 성공하였으나, 이후에 jdbc로 부르는 부분이 먹히지 않는다.  
몇일 째 시도중이지만 개선되지 않아 지금까지 햇던 것들을 정리하려고 한다.  


1) 직접적인 다이렉트 연결
-> private subnet이 아닌 public subnet에 있다면 이상 없이 연결이 된다.

2) SSH shell 활용한 접근 방식
- EC2에서 Mysql에 접근할 때 아래와 같이 접근한다.
```
ssh -i {pem 경로} -N -L 13306:rds-instance:rds-port ubuntu@instance
```

- 터널링에서 13306 -> 3306 으로 포트포워딩된 상태에서 로컬에서 mysql로 접속
```
mysql -h 127.0.0.1 -P 13306 -u userid -p
```
- 하지만 Code Level에서 연결하는 것과는 다른 이야기
위와 같이 하면 command shell에서는 동작한다.

3) SSHTunneling
- Python을 활용하여 터널링을 열어 EC2에 접근하는 방식
- 임의로 특정 포트를 열어 연결
- N옵션을 주어 Local에서도 실행되게끔
- 아래 코드는 SSHTunneling을 활용하여 RDS에서 Command를 날린 것과 같다.
- RDS 자체에서 동작하도록 한 것이다.

다만 해주고 싶은 부분은 pyspark를 통해 바로 건드리고 싶은것이다..  

SSHTunnelForwarder과 jdbc 연결을 통해 접근하여 보았으나, ec2를 거치지 않고 운영되는 rds만 접속이 될 뿐 ec2를 거치는 rds는 아직 부족하다.
```
from sshtunnel import SSHTunnelForwarder
from pyspark.sql import SparkSession
import os

pem_file_dir = os.path.dirname('pem/')
    
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
        remote_bind_address=(rds_hostname, 3306)
    ) as tunnel:
        print('--start ssh--')
        tunnel.start()
        local_binding_port = tunnel.local_bind_port
        print('bind port : ' , local_binding_port)
        db = pymysql.connect( host='127.0.0.1', user=rds_username, password=rds_password, port=local_binding_port, database= rds_database)
        try:
            with db.cursor() as cur:
                cur.execute(f'select * from {rds_table}')
                df = cur.fetchall()
                print(df)
                #for r in cur:
                    #print(r)
        finally:
            db.close()
except BaseException as e:
    print('Problem is --> ', e)
finally:
    if tunnel:
        tunnel.close()

df_spark = spark.createDataFrame(df)
df_spark.show()

```


4) Pyspark read jdbc
- 목표점으로 둔 것이 이 부분인데.. connection refuse 에러부터 로그를 보며 찾아보려고 하였으나, 아직 까지 해결 방안을 찾지 못했다ㅠㅠ
- 아래와 같이 구현하면 될 거 같은데 완성되진 않았으니 참고용

```
try:
    with SSHTunnelForwarder(
        (ssh_hostname, 22),
        ssh_username=ssh_username,
        ssh_pkey=pem_file_path,
        remote_bind_address=(rds_hostname, 3306)
    ) as tunnel:
        print('--start ssh--')
        tunnel.start()
        print('bind port : ' , tunnel.local_bind_port)
        bing_port = tunnel.local_bind_port

        # MySQL에 연결
        jdbc_url = f"jdbc:mysql://localhost:3306/{rds_database}?enabledTLSProtocols=TLSv1.2"
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