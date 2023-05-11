---
layout: post
title:  "Python List tuple 차이"
author: Kimuksung
categories: [ Python ]
tags: [ Python, List, Tuple ]
# 댓글 기능
comments: False
image: "assets/images/python.png"
---


##### What is the difference between list and tuples in Python?
---
정답
- 값이 변경 가능한가? 여부
- 메모리 → 속도 차이

### List
---
- Element 수정 가능
- two blocks of memory ( 하나는 고정 사이즈, 실제 데이터 )

```python
a = [1]

# List to Tuple
tuple((a))
```

##### Tuple
---
- `Update`, `Insert`, `Delete`, `Sort`, `Reverse` 불가능
- Single block of memory

```python
a = (1,)

# Tuple to List
list((a))
```

참조
[https://www.educative.io/answers/tuples-vs-list-in-python](https://www.educative.io/answers/tuples-vs-list-in-python)