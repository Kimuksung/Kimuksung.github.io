---
layout: post
title:  "Python __init__?"
author: Kimuksung
categories: [ Python ]
tags: [ Python ]
image: assets/images/python.png
comments: False
---

Python은 OOP 객체지향 언어입니다.
Class를 지원하면 Object 형태를 나타낼 수 있습니다.
<br/>

##### __init__
---
- 초기화 메서드 = 생성자(constructor)
- 아래 코드와 같이 표현되며, Class Instance는 처음 호출 시 __init__이라는 함수를 호출 합니다.
- Instance = Class Object를 의미

```python
class Human():
	def __init__(self):
		pass
```

##### __del__
---
- 소멸자(destructor)
- Object가 사라지기 전에 처리하기 위함
<br/> 
   
##### __repr__
---
- Representatoin = Object를 표현한다는 의미로써, print 와 같이 사용자가 이해할 수 있는 Object를 나타나기 위함
- Object를 인자로 하여 repr 함수를 실행시켜 Class에 정의된 값을 Return

```python
class Human():
	def __init__(self,name):
		self.name = name

	def __repr__(self):
		return "my name is " + self.name

a = Human("김욱성")
print(a)
> 
my name is 김욱성
```

그렇다면 double underscore가 무슨 의미일까요?

- Name Mangling이라 불리며, private 변수,함수를 만드는데 사용
- 나중에 다른 Class가 상속 할 때, 이 변수를 override하는 것을 막아준다