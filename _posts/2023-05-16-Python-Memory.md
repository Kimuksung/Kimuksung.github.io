---
layout: post
title:  "Python Memory 동작 방식"
author: Kimuksung
categories: [ Python ]
tags: [ Python, Memory, GIL, Pointer, Reference Count, Circuit Reference ]
image: assets/images/python.png
# feature tab 여부
featured: true
# story에서 보이게 할지 여부
hidden: true
# 댓글 기능
comments: False
---

안녕하세요  
오늘은 Python이 메모리를 어떻게 관리하는지에 대해 알아보겠습니다.  
평상시에 Python 언어로 코드를 자주 작성하는데 데이터를 어떻게 처리하고 메모리에 어떤 방식으로 올라가는지 궁금하여 작성합니다.

<br>

##### 프로그래밍 메모리
---
- 일반적으로 프로그래밍 언어에서 변수는 단순히 메모리에 있는 Object 주소에 대한 Pointer  
- Program에서 변수가 사용될 때 Process는 Memory에서 값을 읽어 동작

문제 발생하는 경우
- 메모리 반환 X
- dangling Pointer = 메모리를 너무 일찍 반환하여 사용 중인 메모리에 접근

해결 방안 : 자동 메모리 관리 프로그램 실행

- 메모리를 활용하여 Program의 모든 참조를 Trace
- Reference Count = Object Reference가 0이면 삭제 가능

<br>

##### Python 메모리 관리
---
- 기본 파이썬은 Cpython으로 C프로그래밍 언어로 작성
- 물론 Cpython 이외로도 구현되어 있을 수 있다. ( Pypy, Jython, IronPython )
- Python Code → Translate → Byte Code (.pyc, _pycache)
    
    ![https://ifh.cc/g/swwL1k.png](https://ifh.cc/g/swwL1k.png)
    
<br>

Python Memory 특징  
- Memory 관리 = **`Reference Count`** + **`Detect Circuit reference`**    
- Reference Count  
    - Python은 모든 것이 Object ( int, str .. ) → PyObject라는 개념을 사용
        - `ob_refcnt` : 참조 수 + `ob_type` : 다른 유형의 포인터

위와 같은 특징 때문에 작은 메모리 할당이 필요하다.

즉, 변수는 Memory에 있는 **`Object에 대한 참조`**! → 변수는 값을 포함 X, Memory 주소에 대한 참조

- 메모리 작업의 속도 증가와 단편화를 줄이기 위해 PyMalloc이라는 관리자를 사용
- Reference Count 방식을 도입하여 dict,tuple Object 배치를 Container에 배치 시 reference 값 증가
- Heap 영역에 관리되는 동적 메모리 할당
- Python Memory Manager가 API 기능을 통해 Heap Memory allocate,deallocate 처리
    
    ![https://ifh.cc/g/Xz6L0h.jpg](https://ifh.cc/g/Xz6L0h.jpg)
    

아래 그림과 같이 var2,var3가 같은 Object를 참조한다고 가정하여 보자.

- Multi Thread를 통해서 동시에 접근하게 된다면 어떻게 해야할까?
- GIL(Global Interpreter Lock)을 사용하여 Byte code를 실행 시 GIL을 획득해야 처리 가능하다.
- 위와 같은 이유로 Python Program은 단일 Thread로 처리.

![https://ifh.cc/g/qF8A7l.png](https://ifh.cc/g/qF8A7l.png)

![https://ifh.cc/g/z6z7Oh.png](https://ifh.cc/g/z6z7Oh.png)

순환 참조 감지

- Reference Count가 주로 사용하지만, 보조적으로 Generation GC라는 방법으로 사용
- Object에 참조 횟수는 1이지만, Object에 접근이 불가한 경우
- 순환 참조는 Container Object(Tuple,List,Set,Dict)에서만 발생
- Object가 새롭게 생성된다면 Generation 0에 Linked list로 추가
- Reference 값이 0이 된다면 Linked List에서 해제한다.
- 결국 Gargabge Collection이 아래와 같은 방식으로 처리하려고 하다보면, Object가 많아질수록 Memory에 있는 Object를 N개를 탐색하여 일일히 확인해야하기 때문에 시간도 오래 걸리고 주기가 길어진다.

![https://ifh.cc/g/z30lHH.png](https://ifh.cc/g/z30lHH.png)

![https://ifh.cc/g/JnTTcZ.png](https://ifh.cc/g/JnTTcZ.png)

![https://ifh.cc/g/01OZJh.jpg](https://ifh.cc/g/01OZJh.jpg)


###### 참조

- [https://kimwooseok.com/python/2022/07/25/python_memory_management/](https://kimwooseok.com/python/2022/07/25/python_memory_management/)