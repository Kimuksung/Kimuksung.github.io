---
layout: post
title:  "Python path"
author: Kimuksung
categories: [ Python ]
tags: [ Python, sys.path ]
image: assets/images/python.png
comments: False
---

<br>

#### PYTHON PATH란?

---
import를 통해 package,module을 불러오는 경우 Python은 내부적으로 파일을 찾기 위해 **`sys.path`**와 **`PYTHONPATH`**에 있는 경로를 탐색

##### sys.path
---
- 실제 코드가 동작하는 위치로 코드 내부에서 sys library를 통해 추가할 수 있다.
- Default = .py 파일이 Directory의 절대 경로
- sys.path.append를 통해 Directory를 추가하여 모듈을 찾을 수 있다.
- 아래 예시에서 /Users/airflow/dags → /Users/airflow/ 순차적으로 module을 찾습니다.
- 찾으면 Import 없으면 리스트 순으로 계속하여 검색 후 없다면 `ModuleNotFoundError` Exception이 발생

```python
import sys
print(sys.path)
> 
['/Users/airflow/dags', '/Users/airflow/', .. ]
```

```python
# /home/ubuntu/test.py
import sys
sys.path.append("/opt")

import common
```

##### PYTHONPATH
---
- 환경 변수로 경로를 추가하면, Python은 Directory 경로들을 sys.path에 추가
- python code에서만 아니라 외부에서도 sys.path 조작 가능

<br>

##### 기타 기본 경로
---
- Python에 포함된 여러 내장 모듈을 탐색하기 위한 경로
- 아래와 같이 zip 압축 파일도 추가 가능

```python
/opt/homebrew/Cellar/python@3.10/3.10.10/Frameworks/Python.framework/Versions/3.10/lib/python310.zip
```

우선 순위

1. .py 파일이 속한 Directory 절대 경로
2. PYTHONPATH 환경 변수
3. 기타 기본 경로

만약 내장 모듈과 같은 이름으로 Local file이 생성된다면 Local file을 우선하여 불러온다.

그러기에 기존 module이 존재하는지 확인하고 명명해야 한다!

<br>