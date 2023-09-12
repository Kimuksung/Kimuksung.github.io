---
layout: post
title:  "Hadoop Cluster 단일 구성하기 with Ec2"
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

#### Java Jdk 8 다운로드
---

- https://www.oracle.com/java/technologies/downloads/#java8-linux

- 연결
```bash
$ ssh -i [pem파일경로] [ec2-user계정명]@[ec2 instance의 public IP 또는 public DNS]
$ ssh -i "~/.ssh/wclub-server.pem" ec2-user@ec2-3-35-49-2.ap-northeast-2.compute.amazonaws.com
```

- 파일 업로드

```bash
$ scp -i [pem파일경로] [업로드할 파일 이름] [ec2-user계정명]@[ec2 instance의 public DNS]:~/[경로]
$ scp -i "~/.ssh/wclub-server.pem" ~/Downloads/jdk-8u381-linux-x64.tar.gz ec2-user@ec2-3-35-49-2.ap-northeast-2.compute.amazonaws.com:~/download
```

JDK 파일을 다운로드 받은 후 Ec2 Instance에 업로드해줄 겁니다.

아래와 같이 진행해주면 됩니다.

```bash
$ scp -i [pem파일경로] [ec2-user계정명]@[ec2 instance의 public DNS]:~/[경로] [다운로드 파일의 로컬 경로]
```

```bash
$ tar -xvf jdk-8u381-linux-x64.tar.gz
```

```bash
$ mv jdk1.8.0_381/ /usr/local/jdk1.8
```

Java 환경 설정하기

- 기본적으로 etc/profile 전역적인 곳에 처리할 예정입니다.
- linux 환경 설정

```bash
$ sudo vi /etc/profile
$ source /etc/profile
```

```bash
# /etc/profile
export JAVA_HOME=/usr/local/jdk1.8
export PATH=$PATH:$JAVA_HOME/bin
export JAVA_OPTS="-Dfile.encoding=UTF-8"
export CLASSPATH="."

# :wq
# w!
```

- 이를 하게 되면, 기존의 bash command가 먹히지 않는다.
- 아래와 같이 임시로 처리해야한다.

```bash
export PATH=%PATH:/bin:/usr/local/bin:/usr/bin
```

```bash
$ cd ~
$ wget https://dlcdn.apache.org/hadoop/common/hadoop-3.2.4/hadoop-3.2.4.tar.gz
$ tar -xvf hadoop-3.2.4.tar.gz
```

```bash
# Add /etc/profile
export JAVA_HOME=/usr/local/jdk1.8
export JAVA_OPTS="-Dfile.encoding=UTF-8"
export CLASSPATH="."
export HADOOP_HOME=/home/ec2-user/hadoop-3.2.4
export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
export HDFS_NAMENODE_USER="root"
export HDFS_DATANODE_USER="root"
export HDFS_SECONDARYNAMENODE_USER="root"
export YARN_RESOURCEMANAGER_USER="root"
export YARN_NODEMANAGER_USER="root"
```

- mapreduce 테스트하기

```bash
$ hadoop jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.2.4.jar wordcount $HADOOP_HOME/etc/hadoop/hadoop-env.sh wordcount_output
$ ls -al wordcount_output/*
$ cat wordcount_output/part-r-00000
$ rm -rf wordcount_output/
```
