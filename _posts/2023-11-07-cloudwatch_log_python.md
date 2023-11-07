---
layout: post
title:  "Python으로 CloudWatch 로깅하기"
author: Kimuksung
categories: [ Python, CloudWatch]
tags: [ Python, CloudWatch]
image: assets/images/cloudwatch.png
comments: False
---

##### 요구 사항
---
만약, 내부 서비스에 대해 누가 사용한지 여부를 로깅하여 컨트롤하고 싶다면 어떻게 해야 할까?

"AWS 클라우드 기반으로 여러 서비스를 이용 중이니 CloudWatch를 활용하여 연동하여 본다."  
라는 생각으로 구성하여보았다.

<br>

##### 클라우드 워치에 로그 만들기
---
- CloudWatch에 파이썬에서 generate한 로그 만들기
- API 혹은 로깅이 필요할 때 AWS Cloudwatch에 구성
- 다양하고 대량의 API를 처리해야할 정도가 온다면, api gateway 고려해야 한다.
- boto3와 watchtower 라이브러리를 사용
- log 사용 시 debug 모드는 노출되지 않는다. 그외의  info, 등 다른 값들은 가능

```python
$ pip install boto3, watchtower
```

<a href="https://ibb.co/gRk4zVT"><img src="https://i.ibb.co/sV8WyHw/2023-11-07-3-05-14.png" alt="2023-11-07-3-05-14" border="0"></a>


```python
import boto3
import watchtower
import logging

class CloudWatch:
    def __init__(self, log_group_name, log_stream_name):
        self.client_region_name = "ap-northeast-2"
        self.aws_access_key_id = ""
        self.aws_secret_access_key = ""
        self.log_group_name = log_group_name
        self.log_stream_name = log_stream_name

    def set_log(self, logs):
        boto3_client = boto3.client("logs", region_name=self.client_region_name, aws_access_key_id=self.aws_access_key_id, aws_secret_access_key=self.aws_secret_access_key)

        handler = watchtower.CloudWatchLogHandler(
            boto3_client=boto3_client,
            log_group_name=self.log_group_name,
            log_stream_name=self.log_stream_name,
        )

        logger = logging.getLogger(__name__)

        # 로그 레벨 설정 (INFO 레벨까지만 전송)
        logger.setLevel(logging.INFO)  # INFO 레벨과 그보다 높은 레벨의 로그만 전송

        formatter = logging.Formatter("%(asctime)s [%(levelname)s] %(message)s")
        handler.setFormatter(formatter)
        handler.setLevel(logging.INFO)

        logger.addHandler(handler)
        logger.info(logs)

cloud = CloudWatch("uk_test", "test")
cloud.set_log("test to cloudwatch")
```

<a href="https://ibb.co/thq36Fp"><img src="https://i.ibb.co/SxtJgGw/2023-11-07-11-37-21.png" alt="2023-11-07-11-37-21" border="0"></a><br /><a target='_blank' href='https://nonprofitlight.com/ny/new-york/garden-of-eden-foundation-inc'></a><br />

##### 참조
---
- https://hello-bryan.tistory.com/534