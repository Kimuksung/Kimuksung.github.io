---
layout: post
title:  "Hadoop Cluster 가상 분산 모드 구성하기 with Ec2"
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

EC2에 가상 분산 모드를 구성해보려고 합니다.

기본적인 설정은 단일 구축하기에 설정해두었습니다.

### ssh 설정하기

- 아래와 같이 ssh을 설치하여 줍니다.

```bash
$ yum install openssh*
```

- ssh 연결을 위해 아래와 같이 설정하여 줍니다.

```bash
$ cd ~/.ssh
$ ssh-keygen -t rsa -P ""
$ cat id_rsa.pub >> authorized_keys
$ ssh localhost
```

### 환경 설정

---

- Hadoop 파일 설정들을 변경해줄 예정입니다.
- 보기 편하게 vi에 set number를 설정해서 보기 편하게 바꾸어줍니다.

```bash
$ sudo vi $HADOOP_HOME/etc/hadoop/hadoop-env.sh
$ set number
```

아래 순서대로 각 파일들에 아래 값들을 추가해줍니다.

```bash
$ sudo vi $HADOOP_HOME/etc/hadoop/hadoop-env.sh

# change this
export JAVA_HOME=/usr/local/jdk1.8

$ sudo vi $HADOOP_HOME/etc/hadoop/core-site.xml
# change this
<configuration>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://localhost:9000</value>
  </property>
</configuration>

$ sudo vi $HADOOP_HOME/etc/hadoop/hdfs-site.xml
# change this 
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>1</value>
  </property>
</configuration>
```

이제부터는 hdfs namenode를 초기화 해줄 차례입니다.

하지만, 아래와 같이 에러가 발생하네요.

지금까지는 접속한 유저로 처리되어서 에러가 난거 같아서 root 유저를 만들 예정입니다.

```bash
$ cd hadoop-3.2.4
$ hdfs namenode -format
> ERROR: namenode can only be executed by root.
```

### root 유저 만들기

---

- 루트 유저를 아래와 같이 만들줍니다.

```bash
$ sudo passwd root

$ sudo vi /etc/ssh/sshd_config
# change
PermitRootLogin yes

$ sudo mkdir /root/.ssh
$ sudo cp /home/ec2-user/.ssh/authorized_keys /root/.ssh
$ ssh -i 'C:\키페어경로\키페어.pem' root@접속IP

# permission denied 발생 시
$ cd ~/.ssh
$ ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
$ ssh localhost
```

### Hadoop 실행하기

---

- 환경 변수 및 ssh 설정이 완료되었으니, 이제부터는 실제로 실행해보아야겠죠?
- 아래와 같이 진행해주면 바로 연동 완료!
- https://stackoverflow.com/questions/48978480/hadoop-permission-denied-publickey-password-keyboard-interactive-warning

```bash
$ hdfs namenode -format
$ cd $HADOOP_HOME/sbin
$ start-dfs.sh
$ start-yarn.sh
```

결과는 아래와 같다.

<a href="https://ibb.co/7XfRWBH"><img src="https://i.ibb.co/DtJ8CmN/Untitled-13.png" alt="Untitled-13" border="0" width=300 heigh=300></a>
<a href="https://imgbb.com/"><img src="https://i.ibb.co/Sdjy1B8/Untitled-9.png" alt="Untitled-9" border="0" width=300 heigh=300></a>
<a href="https://ibb.co/4NjBFF2"><img src="https://i.ibb.co/vzJ2cch/Untitled-10.png" alt="Untitled-10" border="0" width=300 heigh=300></a>
<a href="https://ibb.co/fFXf5Xy"><img src="https://i.ibb.co/pw1c81M/Untitled-11.png" alt="Untitled-11" border="0" width=300 heigh=300></a>