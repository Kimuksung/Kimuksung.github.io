---
layout: post
title:  "Python Case-sensitive?"
author: Kimuksung
categories: [ Python ]
tags: [ Python, Case-sensitive ]
image: assets/images/python.png
comments: False
---

##### Case-sensitive란?
---
- 대소문자를 구별하여 인식하는 경우
- Python은 일반 프로그래밍 언어와 동일하게 대소문자를 구별한다. (java,c++,js)
- 동일한 이름의 변수이더라도 결과는 다르다.

```python
user_name = "123"
User_Name = "1234"
User_name = "12345"

print(user_name)
print(User_Name)
print(User_name)
```

Python에서는 중요하게 사용되는데
일반적인 Keyword = `def`, `import`, `is`, `not`, `return`, `with` 는 소문자로 사용
예외도 존재 = `True`, `False`, `None`

##### 함수
---
- lower() : 모든 문자가 소문자인 문자열을 반환
- upper() : 모든 문자가 대문자인 문자열을 반환
- casefold() : 모든 문자가 소문자인 문자열을 반환한다. 이 메서드는 lower() 메서드와 유사하지만 유니코드가 아닌 문자열도 변환해 준다.