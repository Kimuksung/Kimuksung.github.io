---
layout: post
title:  "Dockerfile"
author: Kimuksung
categories: [ Docker ]
tags: [ Docker ]
image: assets/images/docker.svg
comments: False
---

Dockerfile 구성 시 사용 할 수 있는 명령어를 정리해두었습니다.

CMD, ENTRYPOINT에 대해서는 다른 페이지에서 상세히 다룰 예정입니다.

##### Dockerfile
---
- FROM : Image
- COPY : 로컬 파일시스템에서 복사
- ADD : 로펄 파일 또는 url에서 파일을 다운로드/복사/압축해제
- RUN : 빌드시 명령어로 수행하는 Layer 생성 ( 최대한 적게 )

```bash
FROM ubuntu
Add batch_ps.sh /usr/local/bin/batch_ps.sh
RUN chmod +x /usr/local/bin/batch_ps.sh
ENTRYPOINT ["/usr/local/bin/batch_ps.sh"]
CMD ["5"]
```

```bash
FROM apache/airflow:2.0.0
COPY requirements.txt .
RUN pip install -r requirements.txt
```

Docker Container 구성 시 자동으로 실행할 command 구성

- Entrypoint : 컨테이너 구성 시 인자값 상관 없이 실행하는 command
- cmd : 컨테이너 구성 시 인자값을 같이 넘겨주면 생략