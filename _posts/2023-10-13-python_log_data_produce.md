---
layout: post
title:  "로그 데이터 전처리 및 Kafka Produce 처리하기"
author: Kimuksung
categories: [ Python, Kafka ]
tags: [ Python, Kafka ]
image: assets/images/python.png
comments: False
featured: False
---

앞에서는 가상의 로그 데이터를 발생시켰습니다.

이제부터는 로그 파일에 저장된 데이터들을 읽은 뒤, Json 형태로 변경하여 Kafka에 Produce하려고 합니다. 

이상적인 그림은 실 서비스에서 유저가 사용하는 이벤트에 대해 직접 API 형태로 구성하면 좋겠지만, 개인 세미 프로젝트 형식으로 진행하기에 이를 로그 파일에 저장한 뒤 읽어 Produce하는 것으로 대체하였습니다.

##### 요구 사항
---
- 일별 log Generator로 만들어진 Log를 Kafka `log` topic에 Produce
- 로그 파일을 각 Key에 맞추어 생성 및 Dictionary로 변환
- `,` 로 나눌 경우 예외 사항이 별도로 필요하여 `\t` 로 split 처리

##### 진행 사항 및 코드
---
- Kafka Produce, Consume하는 Module을 분리하여 구성
- Logdata 처리 시 양이 많아질 경우 메모리가 터질 수 있으니, generator로 구성
- Logdata 처리 시 Kafka Module을 상속 받아 produce 함수로 구성
- kafka Topic 미리 생성
    
    ```python
    # kafka topic list
    $ bin/kafka-topics.sh \
        --bootstrap-server master:9092,worker:9092.. \
        --list
    
    $ bin/kafka-topics.sh \
        --bootstrap-server master:9092,worker:9092.. \
        --replication-factor 3 \
        --partitions 3 \
        --topic log \
        --create
    ```
    

```python
# kafka_module.py
from kafka import KafkaProducer
from kafka import KafkaConsumer
from json import dumps
from datetime import datetime
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

            producer.flush()
            producer.close()

            print(f'[Produce] 걸린시간: {time.time() - start}')
            return {"status": 200}

        except Exception as exc:
            raise exc

    def get_consume(self, option="earliest"):
        try:
            consumer = KafkaConsumer(
                self.topic,
                bootstrap_servers=self.bootstrap_servers,
                value_deserializer=lambda x: x.decode(
                    "utf-8"
                ),
                auto_offset_reset=option #default = latest / earliest = 과거 데이터도 consume
            )
            print(f'[Consume] - 시작')

            for message in consumer:
                print(f'Partition : {message.partition}, Offset : {message.offset}, Value : {message.value}')

        except Exception as exc:
            raise exc

        finally:
            consumer.close()

if __name__ == "__main__":
    # example
    kafka = KafkaInfos(topic='test', nodes=['ip:port'])

    # produce
    messages = {
        'date': datetime.now(),
        'id': '12345',
        'status': 200
    }

    # produce
    kafka.set_produce(messages)

    # consume
    kafka.get_consume()
```

```python
# log_data_json_produce.py
from kafka_module import KafkaInfos

class LogData(KafkaInfos):
    def __init__(self, logfile_dir, logfile_name):
        self.logfile = logfile_dir+logfile_name
        self.__bootstrap_servers = ['master:9092', 'worker1:9092', ..]
        self.__topic = "log"
        super().__init__(self.__topic, self.__bootstrap_servers)

    def parsing_read(self):
        with open(self.logfile) as f:
            f = f.readlines()

            for line in f:
                userid, ip, log_time, method, status, url, agent, *rest = line.split('\t')
                json_log = {
                    'userid': userid,
                    'ip': ip,
                    'log_time': log_time,
                    'method': method,
                    'status': status,
                    'url': url,
                    'agent': agent,
                }
                yield json_log

    def produce(self):
        self.set_produce(self.parsing_read())

if __name__ == "__main__":
    logfile_dir = "/Users/"
    logfile_name = "/access_log_20231013.log"

    logobject = LogData(logfile_dir, logfile_name)

    # generator
    print(logobject.parsing_read())
    # log file read
    [print(i) for i in logobject.parsing_read()]

    # kafka produce
    logobject.produce()
```

##### 결과
---
- 1000개의 데이터가 Produce 된 것을 알 수 있습니다.
<a href="https://ibb.co/N32Gn9s"><img src="https://i.ibb.co/9vHd3bN/2023-10-14-2-11-33.png" alt="2023-10-14-2-11-33" border="0"></a><br /><a target='_blank' href='https://imgbb.com/'>png t</a><br />