---
layout: post
title:  " Python Namespace "
author: Kimuksung
categories: [ Python ]
tags: [ Python, Namespace ]
image: assets/images/python.png
# 댓글 기능
comments: False
---

##### Namespace란?

---

- 특정한 Object를 이름에 따라 구분할 수 있는 범위
- Python은 모든 것들을 Object로 관리한다고 말씀드렸는데,
- 특정 이름과 Object를 Mapping 관계를 **`Dictionary`** 형태로 가지고 있다.

우선 순위 built-in > global > local

1. Global Namespace
    
    모듈 별로 존재
    
    ```python
    test = "kai_test"
    
    print(globals())
    >
    {
    '__name__': '__main__',
    '__doc__': None, 
    '__package__': None, 
    '__loader__': <_frozen_importlib_external.SourceFileLoader object at 0x100940eb0>, 
    '__spec__': None, 
    '__annotations__': {}, 
    '__builtins__': <module 'builtins' (built-in)>, 
    '__file__': '/Users/wclub/git/airflow/wclub-airflow/dags/test6.py', 
    '__cached__': None, 
    'test': 'kai_test'
    }
    ```
    
2. Local Namespace
    
    Function, Method 별로 존재, Funcion 내의 Local variable이 소속
    
    ```python
    test = "kai_test"
    
    def namespace_test():
        test = "function test"
        print(locals())
    
    namespace_test()
    >
    {'test': 'function test'}
    ```
    
3. Built-in Namespace
    
    기본 내장 함수 및 예외들의 이름 저장
    
    ```python
    print(dir(__builtins__))
    ```
    

![https://ifh.cc/g/CTRxAd.jpg](https://ifh.cc/g/CTRxAd.jpg)

##### Global()

---

- Global 변수에 접근 가능하게 만들어준다.

```python
def namespace_test():
    global test
    test = "local_space"
    return

test = "kai_test"
print(test)
namespace_test()
print(test)
```

##### Local()

---

- 상위 Level의 Local 변수를 접근 가능하게 만들어준다.

```python
def namespace_test():
    test = 0
    def nonlocal_test():
        test = 100
        print(test)

    nonlocal_test()
    print(test)

def namespace_test2():
    test = 0
    def nonlocal_test():
        nonlocal test
        test = 100
        print(test)

    nonlocal_test()
    print(test)

namespace_test()
print("-"*10)
namespace_test2()
```

참조

[https://leffept.tistory.com/431#:~:text=네임스페이스란 특정한 객체,을 네임스페이스라고 합니다](https://leffept.tistory.com/431#:~:text=%EB%84%A4%EC%9E%84%EC%8A%A4%ED%8E%98%EC%9D%B4%EC%8A%A4%EB%9E%80%20%ED%8A%B9%EC%A0%95%ED%95%9C%20%EA%B0%9D%EC%B2%B4,%EC%9D%84%20%EB%84%A4%EC%9E%84%EC%8A%A4%ED%8E%98%EC%9D%B4%EC%8A%A4%EB%9D%BC%EA%B3%A0%20%ED%95%A9%EB%8B%88%EB%8B%A4).