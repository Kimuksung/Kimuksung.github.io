---
layout: post
title:  "Hadoop Cluster 클러스터 구성하기(완성) with Ec2"
author: Kimuksung
categories: [ hadoop ]
tags: [ hadoop ]
# 댓글 기능
comments: False
# feature tab 여부
featured: False
# story에서 보이게 할지 여부
image: assets/images/hadoop.jpeg
---

Hadoop 클러스터를 EC2를 활용하여 구성 중에 있습니다.

전편에서는 각 Instance 별로 구성은 완료하였으나, 각 Node가 서로를 인지하지 못하며 정상적으로 작동하지 않아 하나씩 해결해 나갈 예정입니다.

각 문제를 정의하고 어떻게 해결해 나갔는지 정리해보려고 합니다.

<br>

#### 문제
---
1. 재부팅 하거나 로그아웃 후 ssh 연결 이후에 root 유저로 접속하는 경우 /etc/profile에 있는 값들이 적용 안되는 현상
2. HDFS core-site.xml 설정에서 localhost:9000으로 설정 시만 연결 + 이 경우에는 datanode 들을 인지하지 못함
3. HDFS NamdeNode Binding Error
4. Secondary Node Instance를 제대로 구성하지 못함
5. hdfs 재시작 시 datanode들을 제대로 구성하지 못함

<br>

#### 접근 방안
---
##### 문제 1) root 유저로 접속하는 경우 /etc/profile 환경 변수를 가져오지 못함

- sudo su를 사용하게 되면, 환경 변수를 수동으로 적용해주어야 한다.

```bash
$ sudo su
$ source /etc/profile

```

- sudo -
    - root로 전환 및 root 사용자의 환경을 새로 로드하여 /etc/profile이 반영된다.

```bash
$ sudo su -
```

- sudo -i
    - root로 전환하나, 현재 로그인한 환경을 그대로 가져간다.

```bash
$ sudo su -i
```

<br>

##### 문제 2) HDFS core-site.xml 설정에서 localhost:9000으로 설정 시만 연결
- master라는 값이 있음에도 인지하지 못하여 임시적으로 localhost로 구성
- 하지만, datanode를 인지하지 못하는 문제가 생긴다.
- 해결 과정은 문제3번을 참고

<br>

##### 문제 3) HDFS core-site.xml 설정에 master:9000을 설정하면 에러 발생
- 로그 확인 시 java bind exception 에러가 발생
- /etc/hosts의 master ip 값이 public ip로 구성
- AWS VPC에서 binding하기 위해서는 public ip가 아닌 **`private ip`**로 구성해야 한다.
- 참고 - https://stackoverflow.com/questions/42383497/hadoop-config-on-ec2-why-public-dns-works-but-not-public-ip

```bash
# log 보기
cat logs/hadoop-root-namenode-master.log

# 해결 방안
$ vi /etc/hosts
# master public ip -> master private ip
```

    <a href="https://ibb.co/FszZ3B3"><img src="https://i.ibb.co/Q8MSQ6Q/Untitled-26.png" alt="Untitled-26" border="0"></a>

<br>

##### 문제 4) Secondary Node Instance를 제대로 구성하지 못함

- 로그 확인 시 에러가 발생
- 문제 3번과 유사한 결로 worker01의 private ip를 수정처리 한다.

```bash
$ sbin/start-all.sh
# log 보기
cat logs/hadoop-root-secondarynamenode-worker01.log

# 해결 방안
$ vi /etc/hosts
# worker01 public ip -> worker01 private ip
```

    <a href="https://ibb.co/jz3rKBK"><img src="https://i.ibb.co/MfPCTWT/Untitled-27.png" alt="Untitled-27" border="0"></a>

<br>

##### 문제 5) hdfs 재시작 시 datanode들을 제대로 구성하지 못함

- 앞서 진행한 과정들로 인하여 datanode가 미리 생성되어 있다.
- 이미 발생한 값으로 인하여 id를 잘 추적하지 못하기에 아래와 같이 삭제처리하여준다.

```bash
$ cd ~/hdfs_dir
$ rm -rf datanode yarn
$ sbin/start-all.sh
```

    <a href="https://ibb.co/jhPFF5q"><img src="https://i.ibb.co/2q1BB6R/2023-09-15-12-03-45.png" alt="2023-09-15-12-03-45" border="0"></a>
    <a href="https://ibb.co/Yh1ggXb"><img src="https://i.ibb.co/ZxF99SN/2023-09-15-12-11-09.png" alt="2023-09-15-12-11-09" border="0"></a>

<br>

##### 결과
---
- 각 노드 별로 구성 여부
- NameNode, Secondary NameNode, DataNode 가 잘 구성되었는지 확인해봅니다.
```bash
$ jps
$ bin/hdfs dfsadmin -report
```

    <a href="https://ibb.co/NxH8yJv"><img src="https://i.ibb.co/Mn3YCtv/2023-09-15-1-51-01.png" alt="2023-09-15-1-51-01" border="0"></a>
    <a href="https://ibb.co/fMwFQb1"><img src="https://i.ibb.co/JnJtCNB/2023-09-15-1-52-09.png" alt="2023-09-15-1-52-09" border="0"></a>

    <a href="https://ibb.co/zFmtzkv"><img src="https://i.ibb.co/NrFGM4B/Untitled-28.png" alt="Untitled-28" border="0"></a>
    <a href="https://ibb.co/xLkshhr"><img src="https://i.ibb.co/4tqfWW0/Untitled-29.png" alt="Untitled-29" border="0"></a>
    <a href="https://ibb.co/VwnhYWW"><img src="https://i.ibb.co/PDXbGZZ/Untitled-30.png" alt="Untitled-30" border="0"></a>

```bash
$ bin/yarn node -list
```

    <a href="https://ibb.co/jVBnpHv"><img src="https://i.ibb.co/Q8zy5XF/Untitled-31.png" alt="Untitled-31" border="0"></a>

<br>

##### 참고
---
- https://1mini2.tistory.com/104
- https://parksuseong.blogspot.com/2019/04/312-1-standalone-pseudo-distributed.html