---
layout: post
title:  "가상 로그 데이터 생성하기"
author: Kimuksung
categories: [ Python ]
tags: [ Python ]
image: assets/images/python.png
comments: False
featured: False
---

로그 데이터 전체적으로 구성하기 위해서는 로그 데이터가 필요합니다.

별도로 어디서 가져올 수는 없어서 가상의 로그를 만들어보았습니다.

##### 목적
---
- 가상의 로그 데이터를 만들어 Kafka에 Produce하기 위함
- apache faker를 가져다 사용 - `https://gitlab.com/kevintvh/Fake-Apache-Log-Generator`
- 원하는 용도에 맞추어 코드를 커스터마이징하여 사용
    - userid
    - ip
    - event_time
    - http_method
    - response
    - url
    - useragent
- 로그 주기는 1~100초 사이 랜덤 발생
- Output File은 access_log_{YYYYMMDD} 형태
    
    ```python
    $ python log_generator.py -n 1000 -o LOG 
    ```
    

```python
# https://gitlab.com/kevintvh/Fake-Apache-Log-Generator
# python log_generator.py -n 10 -o LOG
import time
import datetime
import numpy
import random
import argparse
import uuid
from faker import Faker
from tzlocal import get_localzone

parser = argparse.ArgumentParser(__file__, description="Fake Apache Log Generator")
parser.add_argument("--output", "-o", dest='output_type', help="Write to a Log file, a gzip file or to STDOUT",
                    choices=['LOG', 'GZ', 'CONSOLE'])
parser.add_argument("--num", "-n", dest='num_lines', help="Number of lines to generate (0 for infinite)", type=int,
                    default=1)
parser.add_argument("--prefix", "-p", dest='file_prefix', help="Prefix the output file name", type=str)
parser.add_argument("--sleep", "-s", help="Sleep this long between lines (in seconds)", default=0.0, type=float)

args = parser.parse_args()

# file_info
log_lines = args.num_lines
file_prefix = args.file_prefix
output_type = args.output_type

faker = Faker()

# time
local = get_localzone()
timestr = time.strftime("%Y%m%d")
otime = datetime.datetime.now()

outFileName = 'access_log_' + timestr + '.log'

f = open(outFileName, 'a')

# log infos
response = ["200", "404", "500", "301"]
http_methods = ["GET", "POST", "DELETE", "PUT"]
useragentlist = [faker.firefox, faker.chrome, faker.safari, faker.internet_explorer, faker.opera]

while log_lines:
    if args.sleep:
        increment = datetime.timedelta(seconds=args.sleep)
    else:
        increment = datetime.timedelta(seconds=random.randint(1, 100))
    otime += increment

    userid = uuid.uuid4()
    ip = faker.ipv4()
    dt = otime.strftime('%Y-%m-%d %H:%M:%S')
    timezone = datetime.datetime.now(local).strftime('%z')
    http_method = numpy.random.choice(http_methods, p=[0.6, 0.1, 0.1, 0.2])

    resp = numpy.random.choice(response, p=[0.9, 0.04, 0.02, 0.04])
    uri_data = faker.uri()
    useragent = numpy.random.choice(useragentlist, p=[0.5, 0.3, 0.1, 0.05, 0.05])()
    logs = [userid, ip, dt+timezone, http_method, resp, uri_data, useragent]
    logs = list(map(str, logs))
    f.write("\t".join(logs)+"\n")

    log_lines = log_lines - 1
    if args.sleep:
        time.sleep(args.sleep)
```