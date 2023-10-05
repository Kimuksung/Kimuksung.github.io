---
layout: post
title:  "Python 답게 생각하기"
author: Kimuksung
categories: [ Python, PEP8 ]
tags: [ Python, PEP8 ]
image: assets/images/python.png
comments: False
featured: True
---

Python 코드의 기술 책을 읽으면, Python 답게 코드를 작성하는 방법을 작성합니다.

#### PEP8

##### 공백
---
- tab 대신 spacebar를 사용
- 중요한 문법 시 들여쓰기는 4*space
- Line 길이는 79개의 이하의 문자
- 여러줄로 작성 시 들여쓰기를 추가하여 작성
- fucntion과 class 작성 시에는 2줄을 띄어쓰기
- Class의 Instance Method끼리는 1줄을 띄어쓴다.
- dictinoary에서는 `key`와`:` 사이에서는 공백 X, 한줄 안에 작성 시 `,` 뒤에 spacebar
- 변수 대입 전후로는 spacebar를 하나만 사용
- type 표기를 하는 경우 `val`과 `:` 사이에는 공백X, `:` 과 `type` 사이에는 공백

##### 명명 규약
---
- function, value, attribute는 소문자와 밑줄(snake_case)
- public instance attribute에는 `_` 을 붙여준다.
- private instance attribute에는 `__` 을 붙어준다.
- class는 CamelCase
- Module 수준의 상수는 모두 대문자와 `_` 를 혼용
- class의 instance method의 첫 parameter는 반드시 `self`
- class의 class method의 첫 parameter는 반드시 `cls`

##### 식과 문
---
- 긍정식을 부정하기보다는 부정을 내부에 넣는다. if not a is b → if a is not b
- 빈 컨테이너, sequence 검사 시 0과 비교하지 않는다. if list 와 같이 사용
- 한줄 짜리 if, for, while, except는 명확성을 위해서 나누어 사용
- 식을 한줄에 쓸 수 없는 경우 괄호로 둘러싸고 줄바꿈과 들여쓰기를 넣어준다.
- 여러 줄에 쓰기 위해서는 괄호를 사용한다.

##### Import 시
---
- import a, from a import b 는 파일의 맨 앞에 위치
- 절대적인 이름을 사용하며, 경로의 상대 이름은 사용하지 않는다.
- 불가피하게 상대 경로를 사용하게 될 경우에는 반드시 from a import foo 처럼 명시적으로 사용한다.
- 우선 순위는 표준 라이브러리 > thirdb party 라이브러리 > 만든 Module 순으로 적어준다.

##### Format 보다는 f문자열 인터플레이션을 사용
---
- 형식 문자열은 C언어에서 넘어왔기에 Python 스럽지 않은 구문이 다수 존재
- 형식 지정자 %d, %s, %f 는 코드 구성 시 헷갈리게 만든다.
- Format을 통하여 구성하여도 위와 같은 문제가 여전히 존재
- 이를 해결하기 위하여 Python에서는 F-문자열을 지원

```python
str = f"{a} = {b}"
```

##### 복잡한 식 보다는 도우미 함수를 사용한다.
---
- Python 스럽게 → DRY = 반복하지 않도록 구성한다.
- E.g) dictionary 값 출력 시 없는 key 예외 처리하는라고 if else 사용 보다는 get 함수를 활용

```python
# 1
my_values = {"red" : 1, "blue" : 2}
my_values.get('key',[''])[0] or 0

# 2
values_str = my_values.get('key', [''])
result = int(values_str) if values_str[0] else 0

# 3
def get_result(values, key, default=0):
    values_str = my_values.get('key', [''])
    if value_str[0]:
        return int(values_str)
    return default
```

##### Unpacking을 활용하라
---
- Index를 사용하는 대신 대입을 사용한다.
- 불필요하게 변수를 index로 구성하여 적용하지 않아 직관적이다.
- Unpacking = 여러 값을 대입할 수 있는 기능

```python
item = ('a', 'b')
# unpacking
first, second = item

# unpacking
for idx, (name, salary) in enumerate(employee, 1):
    pass
```

##### range()보다 enumerate()를 사용해라
---
- enumerate는 iterator를 lazy generator로 감싸준다.

```python
lst = ['a', 'b']
for i in range(len(lst)):
    pass

# enumerate
for i, value in enumerate(lst):
    pass
```

##### 여러 Iterator를 나란히 수행하려면 zip()을 사용하라
---
- zip은 둘 이상의 iterator를 lazy generator로 묶어준다.
- iterator의 크기가 같지 않다면, `itertools` 내장 모듈인 `zip_longest`를 사용

```python
from itertools import zip_longest

l1 = ['a', 'b']
l2 = [1, 5]

for l1_val, l2_val in zip(l1, l2):
    pass

l3 = [3, 5, 8]
for l1_val, l3_val in zip_longest(l1, l3):
    pass
```

##### `for`, `while` 뒤에 else를 사용하지 말아라
---
- for else 구문은 기존의 로직의 흐름과 다르게 흘러간다.
- for문이 break하지 않는 이상 else 로직을 실행하기 때문에 혼동을 준다.
- 예외 처리를 하고 싶다면, 함수로 구성하여 assert로 빼는것이 오히려 좋다.

```python
for x in ['a']:
    pass
else:
    print('test')

def check(value, lst):
    for value in lst:
        return True
    return False

if check(x, lst):
    assert True
else:
    assert False
```

##### 대입식 반복을 피하라 with 왈라스 표현식
---
- Python에서는 왈라스 표현식을 지원(`:=`)
- 대입 후 동일한 값을 사용하기 보다 한문장으로 깔끔하게 구성

```python
def make_ade():
    pass

def out_of_stock():
    pass

count = fresh.get('레몬', 0)
if count > 2:
    make_ade()
else:
    out_of_stock()

# warlus 표현식
if (count := fresh.get('레몬', 0)) > 2:
    make_ade()
else :
    out_of_stock()
```

```python
fresh_fruit = pick_fruit()
while fresh_fruit :
    for i in fresh_fruit:
        pass
    fresh_fruit = pick_fruit()

# warlus
while fresh_fruit := pick_fruit():
    for i in fresh_fruit:
        pass
```