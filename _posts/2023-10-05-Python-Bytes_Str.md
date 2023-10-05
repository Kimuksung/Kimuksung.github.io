---
layout: post
title:  "Python Bytes vs Str"
author: Kimuksung
categories: [ Python ]
tags: [ Python, Bytes, Str ]
image: assets/images/python.png
comments: False
featured: False
---

문자열 시퀀스 데이터는 2가지 유형으로 나누어집니다.

Bytes와 Str로 나누어지며, 이에 대해 알아보려고 합니다.

Bytes와 Str은 서로 호환이 되지 않습니다.

그러기에 사용 시 주의가 필요합니다.

##### Bytes
---
- ASCII로 Encoding이 필요
- 사람의 눈으로 알아보기 어렵다.
- 8Bytes Sequence로 구성
- 유니코드 데이터를 Encoding 해야 Bytes로 구성 가능
- 파일 구성 시 b를 넣어서 구성해줘야 한다.

```python
# initializing string 
str_1 = "Join our freelance network"

str_1_encoded = bytes(str_1,'UTF-8')
```

##### Str
---
- 유니코드로 구성
- 일반적으로 알고 있는 UTF-8

```python
# initializing bytes
bytes = b'Hello world, Python'

result = bytes.decode('utf-8')
```