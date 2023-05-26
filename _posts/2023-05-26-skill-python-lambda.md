---
layout: post
title:  "Python lambda 함수란?"
author: Kimuksung
categories: [ Python ]
tags: [ Python ]
image: assets/images/python.png
comments: False
---

##### Lambda function
---
- runtime에 생성해서 사용 할 수 있는 익명 함수
- 익명 함수 = 함수명을 명명하지 않고 사용가능한 함수
- 필요한 곳에서 즉시 사용 후 버릴 수 있어 **코드가 간결** + **메모리 절약**

```python
lambda 매개변수 : 표현식
```

```python
def add(x,y):
	return x+y

add(10,20)
> 30

(lambda x,y:x+y)(10,20)
> 30
```