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

#### ssh 설정하기

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

#### 환경 설정

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

#### Ec2 root 설정하기
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

#### Hadoop 실행하기
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

파일 생성

```bash
$ hdfs dfs -mkdir /user
$ hdfs dfs -mkdir /user/root
$ hdfs dfs -mkdir /user/root/conf
$ hdfs dfs -mkdir /input
$ hdfs dfs -ls
```

파일 업로드

```bash
$ mkdir test
$ cd test
$ echo 'hadoop file testing with hello world' > readme.txt
$ hdfs dfs -copyFromLocal /home/ec2-user/test/readme.txt /input

```

Instance 기본 옵션으로 진행하니 서버가 터진다.. 다시 구성할 예정입니다. 

사양을 올려 다시 재진행하니 완료된 것을 볼 수 있습니다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/841a625d-aa66-4b40-81fe-31e8b5793fa0/0528648c-f067-4764-8374-383986a42a56/Untitled.png)

```bash
$ hadoop jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.2.4.jar wordcount /input/readme.txt ~/wordcount-output
$ hdfs dfs -ls ~/wordcount-output
$ hdfs dfs -cat ~/wordcount-output/part-r-00000
```

![스크린샷 2023-09-13 오후 9.27.30.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/841a625d-aa66-4b40-81fe-31e8b5793fa0/1fa86d1d-4d71-43a8-bd7b-42c1320e58d0/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-09-13_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_9.27.30.png)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/841a625d-aa66-4b40-81fe-31e8b5793fa0/b5fed423-aef9-4290-9820-64690ebfbb5a/Untitled.png)