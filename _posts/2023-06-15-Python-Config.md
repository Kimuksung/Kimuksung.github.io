---
layout: post
title:  "Python Config 파일 설정하기"
author: Kimuksung
categories: [ Python ]
tags: [ Python, Config ]
# 댓글 기능
comments: False
image: "assets/images/python.png"
---

상황에 따라 Config 파일을 별도로 만들어두어 관리하는 것이 편리하다.

##### 1. Built-in
---
- 별도의 Python file을 생성하여 관리
- 보안적인 이슈가 존재

```python
# config.py
AWS_CONFIG = {
	'aws_key_id': 'aws_access_id',
	'aws_key': 'aws_access_key'
}
```

```python
# main.py
import config
print( config.AWS_CONFIG['aws_key_id'] )
```

##### 2. 외부 파일을 통한 설정
---
- configparser Library를 활용
- ini, json을 활용하여 설정
    
    ```
    ; config.ini
    [AWS_CONFIG]
    aws_key_id = aws_access_id
    aws_key = aws_access_key
    ```
    
    ```python
    # main_ini.py
    import configparser
    
    config = configparser.ConfigParser()
    config.read('config.ini')
    
    aws_access_id = config['AWS_CONFIG']['aws_key_id']
    aws_access_key = config['AWS_CONFIG']['aws_key']
    
    print( aws_access_id, aws_access_key)
    ```
    
    ```json
    {
        "AWS_CONFIG": {
          "aws_key_id": "aws_access_id",
          "aws_key": "aws_access_key"
      }
    }
    ```
    
    ```python
    # main_json.py
    import json
    
    with open('config.json', 'r') as f:
        config = json.load(f)
    
    aws_access_id = config['AWS_CONFIG']['aws_key_id']
    aws_access_key = config['AWS_CONFIG']['aws_key']
    
    print( aws_access_id, aws_access_key)
    ```
    

##### 3. 동적 로딩
---
- 1번과 2번의 장점을 각각 살려두어 설정 가능
- 원하는 곳에 별도의 파일로 구성 가능하여 다른 Git 설정

```python
# /bin/bash/config.py
AWS_CONFIG = {
	'aws_key_id': 'aws_access_id',
	'aws_key': 'aws_access_key'
}
```

```python
# main.py
import sys
import config

sys.path.append('/bin/bash')

print( config.AWS_CONFIG['aws_key_id'] )
```