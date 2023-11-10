---
layout: post
title:  "Dockerfile CMD vs ENDPOINT"
author: Kimuksung
categories: [ Docker ]
tags: [ Docker ]
image: assets/images/docker.svg
comments: False
featured: True
---

Dockerfile에서 CMD와 ENDPOINT의 차이를 알아본다.

기본적인 Docker 빌드/시작 코드

```bash
$ docker build -t dockername .
$ docker run --name continaer_name dockername
```

*이미지가 꺠지는게 존재하지만 누르면 잘보입니다.

##### CMD
---
- cmd를 실행시키는 방식
- 기본 echo test를 설정한다.

```bash
FROM ubuntu
CMD ["echo", "test" ]
```

<a href="https://postimg.cc/mhTqFX03"><img src="https://i.postimg.cc/mhTqFX03/2023-11-10-1-51-18.png" alt="2023-11-10-1-51-18" width = 1000/></a><br/><br/>

하나의 문자열로 실행시키면 어떻게 될까?
- 하나의 문자열로는 불가능하다.

```bash
FROM ubuntu
CMD ["echo test" ]
```

<a href="https://postimg.cc/7bXR11kF" target="_blank"><img src="https://i.postimg.cc/7bXR11kF/2023-11-10-1-51-52.png" alt="2023-11-10-1-51-52" width = 1000/></a><br/><br/>


변수를 지정
- ENV를 사용하여 dockerfile을 빌드한다.

```bash
FROM ubuntu
ENV VAR1=test_value
CMD echo $VAR1
```

<a href="https://postimg.cc/dL1x57QL" target="_blank"><img src="https://i.postimg.cc/dL1x57QL/2023-11-10-2-04-07.png" alt="2023-11-10-2-04-07" width = 1000/></a><br/><br/>

- 여러 변수를 지정하여 dockerfile을 빌드한다

```bash
FROM ubuntu
ENV VAR1=test_value\
    VAR2=var2
CMD echo $VAR1 $VAR2
```

<a href='https://postimg.cc/kDFm0tds' target='_blank'><img src='https://i.postimg.cc/CM9xfjP3/2023-11-10-2-18-10.png' border='0' alt='2023-11-10-2-18-10' width = 1000/></a>

```bash
$ docker inspect containerid
```

<a href='https://postimg.cc/64GGQLL8' target='_blank'><img src='https://i.postimg.cc/64GGQLL8/inspect-test3.png' border='0' alt='inspect-test3' width = 1000/></a>

옵션을 주어서 실행하게 된다면, **`요청 받은 인자값으로 대체`**

```bash
$ docker run --name containername buildname ps -aef
$ docker inspect containername
```

<a href='https://postimg.cc/xqHCcCVN' target='_blank'><img src='https://i.postimg.cc/CMrfrBv7/inspect-test4.png' border='0' alt='inspect-test4' width = 1000/></a>

##### Entrypoint
---
- ENTRYPOINT도 CMD와 코드 구성은 동일하다.
- 결국 CMD와는 다르게 커맨드에 추가 시 없어진는게 아니라 추가로 동작한다. → 동작원리만 다르다.

```bash
# Entrypoint
FROM ubuntu
ENV VAR1=test_value\
    VAR2=var2
ENTRYPOINT echo $VAR1 $VAR2/
```

```bash
$ docker build -t buildname -f filename .
$ docker run --name containername buildname
$ docker inspect containername
```

<a href="https://postimg.cc/cKwx69Vy" target="_blank"><img src="https://i.postimg.cc/cKwx69Vy/entry.png" alt="entry" width = 1000/></a><br/


- docker 실행시 옵션을 주면 어떻게 될까?
- 커맨드가 추가되는것을 볼 수 있다.

```bash
$ docker run --name containername buildname ps -aef
$ docker inspect containername
```

<a href="https://postimg.cc/R6mMt1B3" target="_blank"><img src="https://i.postimg.cc/R6mMt1B3/entry-command.png" alt="entry-command" width = 1000/></a><br/><br/>

어떤 상황에서 동적으로 둘다 쓸까?
- batch script를 적용해본다.
- 주기적으로 프로세스를 확인하는 batch 작업
- default = 5

```bash
#!/bin/bash

INTERVAL=$1

if [ -z "$INTERVAL" ]; then
  INTERVAL=5
fi

while true;
do
  ps x;
  sleep $INTERVAL;
done
```

```bash
FROM ubuntu

Add batch_ps.sh /usr/local/bin/batch_ps.sh
RUN chmod +x /usr/local/bin/batch_ps.sh
ENTRYPOINT /usr/local/bin/batch_ps.sh
```

```bash
$ docker build -t batch_job -f Batchps .
$ docker run --name batch_test batch_job
```

ENTRYPOINT와 CMD를 사용하여 동적으로 만들기

- 코드도 명확해지고 동적으로 할당이 가능하다.

```bash
FROM ubuntu

Add batch_ps.sh /usr/local/bin/batch_ps.sh
RUN chmod +x /usr/local/bin/batch_ps.sh
ENTRYPOINT ["/usr/local/bin/batch_ps.sh"]
CMD ["5"]
```

```bash
#!/bin/bash

INTERVAL=$1

while true;
do
  ps x;
  sleep $INTERVAL;
done
```

<a href='https://postimg.cc/hzKBvSLT' target='_blank'><img src='https://i.postimg.cc/NMHfNFLJ/batch.png' border='0' alt='batch' width = 1000/></a>