---
layout: post
title:  "Python self(class)"
author: Kimuksung
categories: [ Python ]
tags: [ Python ]
image: assets/images/python.png
comments: False
---

##### Self
---
- self = class intance를 나타나는데 사용
- method = Class의 function
- class 내의 method들은 self라는 parameter 값을 가지게 된다.

이유가 뭘까?
- Python에서는 Instance를 만든 뒤, 첫번째 parameter로 항상 Instance가 전달
- self = Instance  그 자체로 본인을 참조하는 주소 값이다.

그렇기 때문에, Method에 self 값이 없다면 Instance Memory 값을 알 수 없다.  
이를 알아야만, Instance attribute, method 값을 찾아갈 수 있기 때문  
위와 같은 이유 때문에 Foo.func1()은 Foo라는 Object를 선언하자마 접근하기 때문에 접근이 가능한 것이라고 이해하였습니다.  

```python
class Foo:
	def func1():
			print("func 1")
	def func2(self):
			print(id(self))
			print("func 2")

Foo.func1()
print(id(Foo))
a = Foo()
a.func2()
>
func 1
5007251504
4314299168
func 2
```
```python
class Foo:
	def func1():
			print("func 1")
	def func2(self):
			print("func 2")

a = Foo()
a.func1()
>
TypeError: Foo.func1() takes 0 positional arguments but 1 was given
```