---
layout: post
title:  "Python 기본 재정리 with.혼공파"
author: Kimuksung
categories: [ Python ]
tags: [ Python ]
image: assets/images/python.png
comments: False
featured: False
---


혼공파 책을 통하여 Python 다시 한번 기초 정리를 해보았습니다.   
상세한 내용은 넘어갔습니다.

##### 케이스
-----
- 캐멀 케이스
    - 클래스
    
    ```bash
    class KoreanStudent:
    ```
    
- 스네이크 케이스
    - 함수
    - 변수
    
    ```bash
    def sum_student():
    	pass
    ```
    
- 들여쓰기
    - Tab
    - Space * 4
    

##### 자료형
-----
- 정수형
    - 정수형
        - 부동소수점 → 정수형으로 변경 시 에러 발생
        
        ```bash
        int(25.27)
        ```
        
    - 부동소수점 0.527234E2 = 0.527234*10^2
- 문자열
    - format
    
    ```bash
    # 정수형
    :5d = 5글자를 채우는데 빈칸 허용
    :05d = 글자를 채우는데 빈칸은 0
    :+d = 양수, 음수 표현
    : d = 양수면 공백 표현
    :=+5d = 기호를 앞으로 밀기
    #부동소수점
    :15f = 총 15칸 만들기
    :+15f = 부호 추가해서 15칸 만들기
    :+015f = 부호 추가, 빈공간 0으로 추가
    :15.3f = 총 15칸, 소수점은 3자리
    :g = 의미 없는 소수점제거
    ```
    

##### Genereator
-----
- 필요할 때 마다 호출해서 사용해야 한다.
- 이터레이터를 만드는 코드
- Iterator한 Object를 만들어 메모리 효율성을 관리
- reversed(list), enumerate() 전부 generator

##### Iterator
-----
- 메모리의 효율성을 위해 사용
- → List 등을 바로 return 해주지 않고 Iterator Obejct를 전달

##### Function
-----
Function Parameter

- 우선 순위
    - 일반 Parameter > 가변 Parameter( *values ) > default Parameter ( value = 5 )

With()

- 구문이 종료 될 때 알아서 Close() 처리
- 주로 file, database 연결, multiprocessing 등에서 사용

```bash
with open('fie_path', 'w') as file:
	file.write('hello world')
```

__name__

- 프로그램 진입점으로 엔드리 포인트, 메인이라고 불린다.
- 파일을 실행한 위치에서는 `__main__`이다.

__init__

- Package를 읽을 때 어떤 처리를 할지와, 내부 모듈 한번에 가져오는 경우에 추가
- `__all__`과 같이 사용해 가져올 모듈을 지정

```bash
__all__ = ["random", "datetime"]
```

##### 파일
-----
- Text Data
    - 아스키코드, UTF-8 과 같이 사람이 알아볼 수 있는 숫자값으로 Encoding
- Binary Data
    - 이미지, 동영상 데이터와 같이 사람이 알아 볼수 없는 숫자로 Encoding
    - Text data에서 123과 같은 경우는 b123과 같이 효율을 위해 사용하기도 함
    - file 처리 시 `b option`을 추가해주어야 한다.
        
        ```bash
        with open('fie_path', 'wb') as file:
        	file.write('hello world')
        ```
        
    

##### Object
-----
- 속성을 가질 수 있는 모든 것

추상화

- 우리가 아는 사람, 동물, 자동차 등을 모두 표현가능하다.
- 다만, 프로그램적으로 필요한 속성들만 가져와 추상화

Instance

- Class 내에서 Constructor 시 생성

```bash
instance = Class()
```

isinstance()

- instance가 어떤 Class로 부터 만들어졌는지 확인하기 위함
- 상속 관계까지 파악하여 준다.
- 단순한 인스턴스 파악 시에는 `type(instance) == ClassName`으로 처리

```bash
isinstance(a, KoreanStudent)
```

Class 변수

- class에서 사용하는 변수로, instance 변수와 다르다. (init 내에 있는 변수와는 다른 값)

```bash
class KoreanStudent():
	cnt = 0
	def __init__(self, name):
		self.name = name
```

Class 함수

- class에서 직접 접근 가능한 method
- `@classmethod` decorator와 parameter에 `cls`값을 넣어서 구현해야 한다.
- Python은 다른 언어와 다르게 static method도 직접 접근이 가능하다,

```bash
class CustomClass:
    # instance method
    def add_instance_method(self, a,b):
        return a + b

    # classmethod
    @classmethod
    def add_class_method(cls, a, b):
        return a + b

    # staticmethod
    @staticmethod
    def add_static_method(a, b):
        return a + b

CustomClass.add_instance_method(None, 3, 5)
CustomClass.add_class_method(3, 5)
CustomClass.add_static_method(3, 5)
```

```bash
class Language:
    default_language = "English"

    def __init__(self):
        self.show = '나의 언어는' + self.default_language

    @classmethod
    def class_my_language(cls):
        return cls()

    @staticmethod
    def static_my_language():
        return Language()

    def print_language(self):
        print(self.show)

class KoreanLanguage(Language):
    default_language = "한국어"

a = KoreanLanguage.static_my_language()
b = KoreanLanguage.class_my_language()
a.print_language()
# 나의 언어는English
b.print_language()
# 나의 언어는한국어
```

Private & Getter & Setter

- Private 변수는 `__변수이름`으로 지정
- `@Property` Decorator는 getter, setter를 지원
    
    ```bash
    class Employee:
        def __init__(self, name):
            self.name = name
    
        @property
        def name(self):
            return self._name
    
        @name.setter
        def name(self, value):
            self._name = value.upper()
    ```
    

##### 상속
-----
- 부모의 클래스의 속성과 메소드를 가져와 사용
- Override = 부모의 메소드를 변경해서 사용
- `class childclass(parentclass)`로 사용
- `super()` 부모 method를 호출하고 싶은 경우

```bash
class Country:
    name = '국가명'
    population = '인구'
    capital = '수도'

    def show(self):
        print('국가 클래스의 메소드입니다.')

class Korea(Country):
    def __init__(self, name):
        self.name = name

    def show_name(self):
				super.show()
        print('국가 이름은 : ', self.name)
```

##### Garbage Collection
---
- 메모리를 효율적으로 사용하기 위함
- Object로 다루며, 참조 갯수를 통하여 이를 제한
- **변수에 저장하지 않는다면, 바로 파괴자 함수를 호출해서 메모리에서 아웃된다.**