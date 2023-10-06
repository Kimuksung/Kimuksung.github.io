---
layout: post
title:  "Kafka Cluster 클러스터 구성하기 with Ec2"
author: Kimuksung
categories: [ Kafka, AWS ]
tags: [ Kafka, AWS, Ec2 ]
# 댓글 기능
comments: False
# feature tab 여부
featured: True
# story에서 보이게 할지 여부
image: assets/images/kafka.png
---

가상의 대용량의 로그 데이터를 만들어 Kafka → Hadoop에 저장해보려고 합니다.

EC2 인스턴스를 구성 및 접속하는 방법 다른 글에도 많이 나오기에 생략하겠습니다.

카프카 클러스터 구성을 시작해보겠습니다.

<br>

#### 기본 host 설정
---
- 인스턴스에 접속하여 이름을 변경하여 줍니다.
- Cluster 정보를 입력하여 줍니다.

```bash
$ hostname
$ sudo hostnamectl set-hostname kafka1

$ sudo vi /etc/hosts
127.0.0.1 localhost
0.0.0.0 kafka1
13.1.1.1 kafka2
```

#### java 설치
---
- kafka는 java기반으로 동작하기에 java 11을 사용할 예정입니다.
- 모든 인스턴스에 적용하여 줍니다.

```bash
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo apt-get install openjdk-11-jdk

# check install
$ java -version

# delete java
$ sudo apt-get purge openjdk*
```

- 환경 설정을 적용하여 줍니다.

```bash
# 환경 설정
sudo vi ~/.bashrc

#JAVA setting
export JAVA_HOME=$(dirname $(dirname $(readlink -f $(which java))))
export PATH=$PATH:$JAVA_HOME/bin

$ source ~/.bashrc
$ echo $JAVA_HOME
```

#### Zookeeper 설치
---
- Kafka 클러스터 관리를 지원하여 주는 Zookeeper를 설치할 예정입니다.
- 본인 서버 정보에 맞추어 구성하여 줍니다.
- `/var/lib/zookeeper/myid` 각 인스턴스 별로 unique한 정수 값을 넣어줍니다.

```bash
$ wget https://archive.apache.org/dist/zookeeper/zookeeper-3.4.12/zookeeper-3.4.12.tar.gz
$ tar xvf zookeeper-3.4.12.tar.gz
$ cd zookeeper-3.4.12/conf
$ sudo vi zoo.cfg

tickTime=2000
dataDir=/var/lib/zookeeper
clientPort=2181
initLimit=20
syncLimit=5
server.1=kafka1:2888:3888
server.2=kafka2:2888:3888
server.3=kafka3:2888:3888

$ echo 1 > /var/lib/zookeeper/myid
1
$ sudo ./bin/zkServer.sh start

```

```bash
$ sudo vi ~/.bashrc
$ source ~/.bashrc
alias start_zookeeper='/home/ubuntu/zookeeper-3.4.12/bin/zkServer.sh start'
alias stop_zookeeper='/home/ubuntu/zookeeper-3.4.12/bin/zkServer.sh stop'
```

#### kafka 설치
---
- Kafka 버전 별로 이후 동작하는 방식이 다르기에 원하는 버전을 설치하여 주시면 됩니다.
- 저는 3.0.0으로 진행할 예정입니다.

```bash
# version2
$ wget https://archive.apache.org/dist/kafka/2.1.0/kafka_2.11-2.1.0.tgz
$ tar xvf kafka_2.11-2.1.0.tgz

# version3
$ wget https://archive.apache.org/dist/kafka/3.0.0/kafka-3.0.0-src.tgz
$ tar xvf kafka-3.0.0-src.tgz

$ sudo vi config/server.properties
# version3에서는 0이 필수로 필요한가봄
broker.id=1
listeners=PLAINTEXT://:9092
advertised.listeners=PLAINTEXT://kafka1:9092
zookeeper.connect=kafka1:2181,kafka2:2181

# run
$ bin/kafka-server-start.sh -daemon config/server.properties
$ ./gradlew jar -PscalaVersion=2.13.6
# exit
$ bin/kafka-server-stop.sh -daemon config/server.properties

# check
$ cat logs/server.log
$ bin/kafka-topics.sh --version
$ jps

# 버전에 따라서 option이 다른것으로 보인다.
# https://stackoverflow.com/questions/69297020/exception-in-thread-main-joptsimple-unrecognizedoptionexception-zookeeper-is
# version 2.2 이상부터는 zookeeper option 대신 bootstrap-server를 지원
# version 2.2 아래에서는 zookeeper option이 필수

# version2
# create topics
$ bin/kafka-topics.sh --create --zookeeper kafka1:2181 --replication-factor 1 --partitions 1 --topic test
# list topics
$ bin/kafka-topics.sh --list --zookeeper kafka2:2181

# version3
# list topics
$ bin/kafka-topics.sh --bootstrap-server kafka1:9092,kafka2:9092 --list
# describe topics
$ bin/kafka-topics.sh --bootstrap-server kafka1:9092,kafka2:9092 --describe

# create topics
$ bin/kafka-topics.sh --create \
--replication-factor 2 \
--partitions 1 \
--topic test \
--bootstrap-server kafka1:9092,kafka2:9092
# delete topics
$ bin/kafka-topics.sh --bootstrap-server kafka1:9092,kafka2:9092 --delete --topic test
```

- topic 생성을 해보려고 합니다.
- 2.2버전 아래로는 zookeeper가 필수이기에 option 값으로 `zookeeper`를 사용합니다.
- 3.0.0 버전이기에 `bootstrap-server` 옵션으로 topic을 만드어줍니다.

```bash
# 버전에 따라서 option이 다른것으로 보인다.
# https://stackoverflow.com/questions/69297020/exception-in-thread-main-joptsimple-unrecognizedoptionexception-zookeeper-is
# version 2.2 이상부터는 zookeeper option 대신 bootstrap-server를 지원
# version 2.2 아래에서는 zookeeper option이 필수

# version2
# create topics
$ bin/kafka-topics.sh --create --zookeeper kafka1:2181 --replication-factor 1 --partitions 1 --topic test
# list topics
$ bin/kafka-topics.sh --list --zookeeper kafka2:2181

# version3
# list topics
$ bin/kafka-topics.sh --bootstrap-server kafka1:9092,kafka2:9092 --list
# describe topics
$ bin/kafka-topics.sh --bootstrap-server kafka1:9092,kafka2:9092 --describe

# create topics
$ bin/kafka-topics.sh --create \
--replication-factor 2 \
--partitions 1 \
--topic test \
--bootstrap-server kafka1:9092,kafka2:9092
# delete topics
$ bin/kafka-topics.sh --bootstrap-server kafka1:9092,kafka2:9092 --delete --topic test
```

```bash
$ sudo vi ~/.bashrc
$ source ~/.bashrc
alias start_kafka='/home/ubuntu/kafka-3.0.0-src/bin/kafka-server-start.sh -daemon config/server.properties'
alias stop_kafka='/home/ubuntu/kafka-3.0.0-src/bin/kafka-server-stop.sh -daemon config/server.properties'
```

produce&consume

- produce와 consume 과정을 아래 커맨드를 친 뒤에 값을 넣어주면 넘어가는 것을 볼 수 있습니다.
- 아래 Python 응로 구성할 것이기 때문에 자세한 내용은 생략합니다.

```bash
# procude
$ bin/kafka-console-producer.sh --topic test --bootstrap-server kafka1:9092,kafka2:9092

# consume
$ bin/kafka-console-consumer.sh --topic test --bootstrap-server  kafka1:9092,kafka2:9092
```

#### Python에서 Kafka 연결하기
---
- 외부와 연결하기 위해서는 `advertised.host.name` 에 aws ip를 넣어주어야 한다.
- 찾아보니 아래와 같이 변경되었다고 한다.
- The `advertised.port` and `advertised.host.name` configurations were removed. Please use `advertised.listeners` instead.
- https://stackoverflow.com/questions/43565698/connecting-kafka-running-on-ec2-machine-from-my-local-machine
- https://kafka.apache.org/documentation/#brokerconfigs

```bash
$ bin/kafka-server-stop.sh -daemon config/server.properties

$ sudo vi config/server.properties
broker.id=0
listeners=PLAINTEXT://:9092
advertised.listeners=PLAINTEXT://ip:9092
zookeeper.connect=kafka1:2181,kafka2:2181

$ bin/kafka-server-start.sh -daemon config/server.properties
```


#### Python Produce, Consume 구성하기
---
- `bootstrap_servers` : 클러스터 정보
- `auto_offset_reset` : Consume 방식
    - 처음부터 -`earliest`, 현재 이후 부터 consume할지가 결정 `latest`
- https://kafka-python.readthedocs.io/en/master/apidoc/KafkaConsumer.html

```bash
from kafka import KafkaProducer
from kafka import KafkaConsumer
from json import dumps
import time

class KafkaInfos:
    def __init__(self, topic: str, nodes: list):
        self.topic = topic
        self.bootstrap_servers = nodes

    def set_produce(self, values):
        try:
            producer = KafkaProducer(
                bootstrap_servers=self.bootstrap_servers,
                api_version=(2, 5, 0),
                acks=0,
                compression_type='gzip',
                value_serializer=lambda x: dumps(x).encode('utf-8')
            )

            start = time.time()
            print('[Produce] - 메시지 전송을 시작합니다.')

            for i, val in enumerate(values, 0):
                print(f'{i}번 전송중 {val}')
                producer.send(self.topic, value=val)
                # 비우는 작업
                producer.flush()

            producer.close()
            print(f'[Produce] 걸린시간: {time.time() - start}')
            return {"status": 200}

        except Exception as exc:
            raise exc

    def get_consume(self):
        try:
            consumer = KafkaConsumer(
                self.topic,
                bootstrap_servers=self.bootstrap_servers,
                value_deserializer=lambda x: x.decode(
                    "utf-8"
                ),
                auto_offset_reset='earliest' #default = latest / earliest = 과거 데이터도 consume
            )
            print(f'[Consume] - 시작')

            for message in consumer:
                print(f'Partition : {message.partition}, Offset : {message.offset}, Value : {message.value}')

            consumer.close()

        except Exception as exc:
            raise exc

if __name__ == "__main__":
    kafka_connection = KafkaInfos(topic='test', nodes=['public_ip:9092', 'public_ip:9092'])
    # produce
    messages = ['i am', 'ready', 'to test']
    kafka_connection.set_produce(messages)

    # consume
    kafka_connection.get_consume()
```

<a href="https://ibb.co/F82jm7h"><img src="https://i.ibb.co/QYL1pmJ/2023-10-06-4-19-38.png" alt="2023-10-06-4-19-38" border="0"></a>
<a href="https://ibb.co/pQNR11H"><img src="https://i.ibb.co/qChWDDH/2023-10-06-4-40-07.png" alt="2023-10-06-4-40-07" border="0"></a>


#### kafka VPC에 맞추어 연결 값 재설정하기
---
- AWS 구성에 맞추어 내부로 들어오는 로직과 외부에서 오는 로직을 구분
- 테스트 용도로 사용하기 위해 PLAINTEXT로 구성
- https://bagbokman.tistory.com/16

```bash
#before
broker.id=0
listeners=PLAINTEXT://:9092
advertised.listeners=PLAINTEXT://ec2ip-ap-northeast-2.compute.amazonaws.com:9092
zookeeper.connect=kafka1:2181,kafka2:2181

# after
broker.id=0
listeners=INTERNAL://0.0.0.0:19092,EXTERNAL://0.0.0.0:9092
listener.security.protocol.map=INTERNAL:PLAINTEXT,EXTERNAL:PLAINTEXT
advertised.listeners=INTERNAL://ec2-ip.ap-northeast-2.compute.internal:9092,EXTERNAL://ec2-ip.ap-northeast-2.compute.amazonaws.com:9092
zookeeper.connect=kafka1:2181,kafka2:2181
inter.broker.listener.name=INTERNAL

# after
broker.id=2
listeners=INTERNAL://0.0.0.0:19092,EXTERNAL://0.0.0.0:9092
listener.security.protocol.map=INTERNAL:PLAINTEXT,EXTERNAL:PLAINTEXT
advertised.listeners=INTERNAL://ec2-ip.compute.internal:9092,EXTERNAL://ec2ip.ap-northeast-2.compute.amazonaws.com:9092
zookeeper.connect=kafka1:2181,kafka2:2181
inter.broker.listener.name=INTERNAL
```


##### 참조 
---
- [https://amazelimi.tistory.com/entry/Kafka-AWS-EC2에-카프카-클러스터-설치하기-LIM](https://amazelimi.tistory.com/entry/Kafka-AWS-EC2%EC%97%90-%EC%B9%B4%ED%94%84%EC%B9%B4-%ED%81%B4%EB%9F%AC%EC%8A%A4%ED%84%B0-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0-LIM)
- https://blog.voidmainvoid.net/325
- https://co-de.tistory.com/44