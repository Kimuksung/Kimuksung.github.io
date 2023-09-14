---
layout: post
title:  "Hadoop Cluster 클러스터 구성하기 with Ec2"
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

EC2를 활용하여 Hadoop Cluster를 구성 과정을 보여드리려고 합니다.

#### EC2 Instance 설치
---
- AMI = Linux
- Memory = 4GB
- PEM 설정
- Network settings
    - vpc
    - subnet
    - security
- volume
    - root - 8GB
    - store - 30GB

<br>

#### 디스크 포맷하기
---
- 디스크를 분리함으로써, 재부팅 이후에도 디스크 볼륨에 남아있도록 구성
- 아래에 보이는 xvda가 root 디스크, xvdb가 설정해주어야할 디스크이다.

```bash
$ lsblk
```

<a href="https://ibb.co/48M9Xqp"><img src="https://i.ibb.co/m9t12j4/Untitled-18.png" alt="Untitled-18" border="0"></a>

- 마운트 할 대상을 설정
- hdfs_dir로 할 것이다. ( 다른 티스토리를 보고 참고한 내용 )

```bash
$ sudo vi /etc/fstab
/dev/xvdb	/hdfs_dir	xfs	defaults	1	1

$ cat /etc/fstab
$ sudo mkdir /hdfs_dir
$ sudo mount -a
$ df -h
```

<a href="https://ibb.co/bm1zjwb"><img src="https://i.ibb.co/ZXdWR42/Untitled-19.png" alt="Untitled-19" border="0"></a>

<br>

#### Java 1.8 설치하기
---
- Oracle에서 다운로드 받은 jdk1.8을 Instance에 업로드
- 이후 환경 변수 설정

```bash
$ scp -i "~/.ssh/wclub-server.pem" ~/Downloads/jdk-8u381-linux-x64.tar.gz ec2-user@ec2-54-180-102-139.ap-northeast-2.compute.amazonaws.com:~/install_dir/jdk-8u381-linux-x64.tar.gz
$ tar -xvf jdk-8u381-linux-x64.tar.gz
$ sudo mv jdk1.8.0_381/ /usr/local/jdk1.8

$ sudo vi /etc/profile
"
export JAVA_HOME=/usr/local/jdk1.8
export PATH=$PATH:$JAVA_HOME/bin
export JAVA_OPTS="-Dfile.encoding=UTF-8"
export CLASSPATH="."
"

$ source /etc/profile
$ java -version
```

<a href="https://ibb.co/hDJ2fXK"><img src="https://i.ibb.co/Gc4CTsV/Untitled-20.png" alt="Untitled-20" border="0"></a>

<br>

#### Hadoop 설치
---
- Apache hadoop 공식 문서를 들어가 무난해보이는 버전을 하나 골라서 다운로드
- 환경 구성을 통일하기 위해 usr/local 아래에 구성

```bash
$ wget https://dlcdn.apache.org/hadoop/common/hadoop-3.2.4/hadoop-3.2.4.tar.gz
$ sudo tar -xvf hadoop-3.2.4.tar.gz -C /usr/local/
$ sudo chown root:root -R /usr/local/hadoop-3.2.4/
```

<a href="https://ibb.co/HrDNmrw"><img src="https://i.ibb.co/p3vWC3V/Untitled-21.png" alt="Untitled-21" border="0"></a>

<br>

#### 환경 설정
---
- 리눅스는 전역 환경 변수를 설정하기 위해 /etc/profile로 설정
- Ec2 Instance를 통해 들어오게 된다면, 자꾸 풀리는데 이는 알아보고 업데이트 할 예정

```bash
$ sudo vi /etc/profile
$ source /etc/profile
```

```bash
# /etc/profile
export JAVA_HOME=/usr/local/jdk1.8
export HADOOP_HOME=/usr/local/hadoop-3.2.4
export YARN_CONF_DIR=$HADOOP_HOME/etc/hadoop
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
```

Hadoop 환경 설정

- 공통 설정
- core-site.xml
- 로그, 네트워크, I/O, 파일시스템, 압축 등 시스템 설정 파일
- 아래와 같이 master로 구성하게 되면, namenode 구성 시 bind 에러가 발생한다..
- 이 부분도 찾으면 업데이트할 예정 일단은 임시로 localhost로 사용

```bash
$ sudo vi $HADOOP_HOME/etc/hadoop/core-site.xml
```

```bash
# core-site.xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
       #<value>hdfs://master:9000</value>
			 <value>hdfs://localhost:9000</value>
    </property>
</configuration>

```

- HDFS 환경 정보
- hdfs-site.xml

```bash
$ sudo vi $HADOOP_HOME/etc/hadoop/hdfs-site.xml
```

```bash
# hdfs-site.xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>3</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:///hdfs_dir/namenode</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:///hdfs_dir/datanode</value>
    </property>
    <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>worker01:50090</value>
    </property>
</configuration>
```

- yarn-site.xml
- resource manager, node manager 설정 정보

```bash
$ sudo vi $HADOOP_HOME/etc/hadoop/yarn-site.xml
```

```bash
<configuration>
<!-- Site specific YARN configuration properties -->
    <property>
        <name>yarn.nodemanager.local-dirs</name>
        <value>file:///hdfs_dir/yarn/local</value>
    </property>
    <property>
        <name>yarn.nodemanager.log-dirs</name>
        <value>file:///hdfs_dir/yarn/logs</value>
    </property>
    <property>
        <name>yarn.resourcemanager.hostname</name>
         <value>master</value>
    </property>
</configuration>
```

- mapreduce 설정
- mapred-site.xml

```bash
$ sudo vi $HADOOP_HOME/etc/hadoop/mapred-site.xml
```

```bash
<configuration>
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>
</configuration>
```

- 하둡 환경 설정
- hadoop-env.sh

```bash
$ sudo vi $HADOOP_HOME/etc/hadoop/hadoop-env.sh
```

```bash
export JAVA_HOME=/usr/local/jdk1.8
export HDFS_NAMENODE_USER="root"
export HDFS_DATANODE_USER="root"
export HDFS_SECONDARYNAMENODE_USER="root"
export YARN_RESOURCEMANAGER_USER="root"
export YARN_NODEMANAGER_USER="root"
```

Instance → 이미지

---

- 인스턴스 구성을 완료하였으니, 이미지화 시켜서 빠르게 구성하여 본다.
- worker를 3개 만들생각이니, 이미지를 사용하여 빠르게 구성한다.

<a href="https://ibb.co/SvBH1CZ"><img src="https://i.ibb.co/KyWBQv8/2023-09-14-12-21-22-1.png" alt="2023-09-14-12-21-22-1" border="0"></a>

<a href="https://ibb.co/6scC2gW"><img src="https://i.ibb.co/KycvYFr/Untitled-22.png" alt="Untitled-22" border="0"></a>

<br>

#### Master 서버 설정
---
- Master 이름 설정
- Ec2 Instance는 별도의 설정을 하지 않는다면, 재부팅 후 IP가 변동하니 주의

```bash
$ sudo hostnamectl set-hostname master
$ hostnamectl
```

<a href="https://ibb.co/Nxjmy1t"><img src="https://i.ibb.co/Tc8bgKY/Untitled-23.png" alt="Untitled-23" border="0"></a>

- 연결할 서버 설정

```bash
$ sudo vi /etc/hosts

54.180.102.139 master
3.39.9.51 worker01
3.36.71.48 worker02
43.201.111.131 worker03
```

Worker 설정

---

- 서버 이름 설정
- Ec2 Instance는 별도의 설정을 하지 않는다면, 재부팅 후 IP가 변동하니 주의

```bash
$ sudo hostnamectl set-hostname worker01
$ hostnamectl
```

<a href="https://ibb.co/D52YM7S"><img src="https://i.ibb.co/wMm7QKx/Untitled-24.png" alt="Untitled-24" border="0"></a>

- 서버 정보

```bash
$ sudo vi /etc/hosts

54.180.102.139 master
3.39.9.51 worker01
3.36.71.48 worker02
43.201.111.131 worker03
```

<br>

#### SSH-KEY 설정하기
---
- 기본 설정하기

```bash
$ sudo su
$ vi /etc/ssh/sshd_config

:set number
# 40
PermitRootLogin yes

# 66
PasswordAuthentication yes

$ systemctl restart sshd
$ passwd
```

- 키 생성
- 각 node에서 다른 node로 비밀번호 없이 접속 가능하다면 완성

```bash
$ ssh-keygen
$ ssh-copy-id root@master
$ ssh-copy-id root@worker01
$ ssh-copy-id root@worker02
$ ssh-copy-id root@worker03

# 검증
$ ssh master, worker01 .. 
```

<a href="https://ibb.co/qWfxS7L"><img src="https://i.ibb.co/dLq7Tjs/Untitled-25.png" alt="Untitled-25" border="0"></a>

```bash
$ cd $HADOOP_HOME
$ bin/hdfs namenode -format /hdfs_dir

$ bin/hdfs datanode -format /hdfs_dir
```

```bash
$ vi $HADOOP_HOME/etc/hadoop/workers
worker01
worker02
worker03
```

```bash
# log 보는 곳
$HADOOP_HOME/logs/hadoop-root-namenode-master.log

vi /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost6 localhost6.localdomain6
```

localhost로 설정하면 된다..

왜지..master로 하면 왜 안되는건지는 모르겠음

나머지 세부 내용은 다시 정리할 예정

```bash
https://ip:9870
```

<a href="https://ibb.co/tBLp26w"><img src="https://i.ibb.co/MnBk1Hy/2023-09-14-11-10-39.png" alt="2023-09-14-11-10-39" border="0"></a>