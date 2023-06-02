---
layout: post
title:  "Python-Iterator"
author: Kimuksung
categories: [ Python ]
tags: [ Iterator, Iterable ]
image: assets/images/python.png
comments: False
---

##### Iterable
---
- 반복 가능한 Object
- 한번 iterate 할 수 있다.
- `iter` = method에 의해 iterator Object로 변경된다.
- list, dict, set, str, bytes, tuple, range

##### Iterator
---
- 값을 차례대로 꺼낼 수 있는 Object
- Iterable한 Object를 내장 함수 혹은 iterable method로 Object 생성
- dir = Object의 properties and methods 정보 ( 변수 제외 )
- iterator로 선언이 되면 내부 method에 next,iter 가 존재한다.
- `__next__`는 순차적으로 값을 꺼낼 수 있도록 만들어둔 기능
- Iterator가 한번 순회한 다음 또 부르는 경우 `StopIteration` Exception 발생

```python
my_list = ["a", "b", "c", "d"]
my_iter = iter(my_list)
print(type(my_iter))

print(dir(my_iter))
>
['__class__', '__delattr__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__iter__', '__le__', '__length_hint__', '__lt__', '__ne__', '__new__', '__next__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__setstate__', '__sizeof__', '__str__', '__subclasshook__']

my_list_iter = my_iter.__iter__()

print(my_list_iter.__next__())
```