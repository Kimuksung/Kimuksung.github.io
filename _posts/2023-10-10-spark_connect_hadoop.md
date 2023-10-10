---
layout: post
title:  "Spark - Hadoop 연결하기"
author: Kimuksung
categories: [ Spark, Hadoop, Ec2, Docker ]
tags: [Spark, Hadoop, Ec2, Docker]
#blog post image
image: assets/images/emr_spark.png
comments: False
#featured: true
---

Pyspark로 연결 시도 시 datanode에 값이 없다고 한다.

단계 별로 문제가 무엇인지 어떻게 해결할지를 구성해보았습니다.

결론 : ec2 인스턴스의 vpc 설정 값으로 외부와 연결하여 read,write가 가능하도록 추가 해결이 필요하다.

##### 1. 파일 생성 권한 사라지는 문제
---
- hadoop cluster 재시작 후 파일 생성 시 권한이 없다.
- hdfs 명령어로 모든 유저가 변경 가능하도록 권한 부여(특정 파일)

<a href="https://ibb.co/hfb77ky"><img src="https://i.ibb.co/Hd6FF0V/Untitled-43.png" alt="Untitled-43" border="0"></a>

```bash
$ hdfs dfs -chmod 777 /
```

##### 2. Spark에서 Hadoop으로 연결
---
- a) datanode 연결 설정 추가

```bash
$ sudo vi $HADOOP_HOME/etc/hadoop/core-site.xml

# 추가
<property>
    <name>dfs.client.use.datanode.hostname</name>
    <value>true</value>
</property>
```

- b) `Connection TimeOut`
- 일반적으로 대부분의 사람이 같은 Node에서 구성하였는지 연결 시 hdfs://localhost:port으로 시도하였으나, 현재 상황에서는 알맞는 답이 아니다.
- 하지만, 나의 상황에서는 별도의 Node에서 구성
- url : `hdfs://IP:Port`
- path : `/test/txt_dir/`

<a href="https://ibb.co/zZ4STCH"><img src="https://i.ibb.co/sqVFdcJ/Untitled-44.png" alt="Untitled-44" border="0"></a>


```bash
from pyspark import SparkConf, SparkContext

conf = SparkConf().setAppName("ReadHDFS")
sc = SparkContext(conf=conf)

hdfs_server = "hdfs://3.34.179.116:9000"
hdfs_file_path = "/test/123/"  # 사용자 이름을 본인의 HDFS 사용자 이름으로 대체하세요.

data = sc.textFile(hdfs_server+hdfs_file_path+"sample.txt")
first_lines = data.take(5)
for line in first_lines:
    print(line)
```

##### 3. 연결이 된 이후에 데이터를 가져올 때 Block 정보를 아는 DataNode가 없다고 한다.
---
- `Connection Timeout`
- 데이터 요청 시 Spark → NameNode → DataNode → 직접 연결할 것으로 예상
- 하지만, Hadoop Cluster 구성 시 private-ip로 서로를 바라본다. (동일 VPC에 존재하는 EC2 인스턴스이기 때문에)
- 그렇기에 NameNode에서 DataNode 값을 전달해줄 때, private-ip를 전달해주어 연결이 불가하여 connection 문제가 발생하는거 같은데.. 이 부분에 대해서는 조금 더 찾아봐야 겠다.
- DataNode를 Public으로 구성하는 경우 인스턴스 시작 시 binding 에러로 구성이 불가하다.

<a href="https://ibb.co/Xk80QqQ"><img src="https://i.ibb.co/PzQXLdL/2023-10-10-11-00-49.png" alt="2023-10-10-11-00-49" border="0"></a>

```bash
$ nc -v ip port
```

##### 참고
---
- https://engineering.linecorp.com/ko/blog/data-engineering-software-troubleshooting