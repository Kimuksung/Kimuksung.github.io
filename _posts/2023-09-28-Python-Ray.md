---
layout: post
title:  "Python MultiProcessing - Ray"
author: Kimuksung
categories: [ Python, MultiProcessing ]
tags: [ Python, MultiProcessing ]
image: assets/images/python.png
comments: False
featured: False
---

- Python에서 병렬처리를 하기 위함
- 쏘카와 AWS Glue에서도 점점 Python Ray 병렬처리 용도로 많이 하여 어떤것인지 궁금하여 조사해보았습니다.

<br>

##### Ray
---
- 일반적으로 파이썬 병렬처리시에는 멀티프로세싱을 주로 사용합니다.(멀티쓰레드는 I/O, Sleep과 같은 CPU 연산이 없는 경우에)
- 위의 상황 마저도, 1억번의 연산정도가 거치지 않는 이상은 Serial한 파이썸 연산이 더 성능이 좋습니다.
- Ray는 Multiprocessing의 pool과 Process와는 다르게 코드로 구현하기가 매우 간단합니다. → 데코레이터만 추가해주면 된다.
- 테스트해본 단계에서는 속도와 메모리의 효율이 잘 나오는지?는 모르겠다.

##### Task
- Function, Class를 Remote 함수로 관리하여 여러 프로세스에서 실행시킬 수 있도록 구성
- 호출 시 Task는 Async 실행하며, .remote()를 통해 Future Object를 Return
- ray.get(Future Object)로 Task를 실행
- ray.put(Object)를 통하여 공통으로 사용할 변수를 모두가 접속 가능한 메모리에 올려둔다. ( Immutable ) → 일반적으로 데이터 프레임을 사용한다고 한다.

```bash
# 함수 구성 시
@ray.remote
def function():
	return True

# 함수 호출
function.remote()

# Return Value
ray.get(function.remote())
```

##### 설치 동작
---
- 여러 블로그를 참조하였으나, 과거 버전이랑 많이 바뀌었는지 맞는 내용이 없었다.
- 아래와 같이 해주면 대시보드를 볼 수 있다. ([http://127.0.0.1:8265](http://127.0.0.1:8265/))

```bash
$ pip install -U 'ray[default]'

# MAC에서 나는 에러로
# https://github.com/urllib3/urllib3/issues/3020
# https://stackoverflow.com/questions/76187256/importerror-urllib3-v2-0-only-supports-openssl-1-1-1-currently-the-ssl-modu
$ pip uninstall urllib3
$ pip install 'urllib3<2.0'

$ ray start --head --include-dashboard=true
```

<a href="https://ibb.co/yPYXFpW"><img src="https://i.ibb.co/4tY7R2P/Untitled-2.png" alt="Untitled-2" border="0"></a><br /><a target='_blank' href='https://usefulwebtool.com/convert-text-to-binary'></a><br />


<a href="https://ibb.co/1nsqv5F"><img src="https://i.ibb.co/X4ys2gc/Untitled-2.png" alt="Untitled-2" border="0"></a><br /><a target='_blank' href='https://usefulwebtool.com/convert-text-to-binary'></a><br />

##### Ray 동작
---
- 함수 혹은 클래스에 `@ray.remote` 를 붙여주면 된다.
- function.remote()와 같은 형태로 호출하면 된다.
- ray.put을 하게 되면 공유하여 쓸 변수를 공통 메모리에 올려준다.
- ray.get은 프로세스들을 실행시켜 값을 기다려 처리하는 곳이다.(비동기)
- ray.wait는 각 프로세스가 동작 완료하면 그때그때 처리할 수 있도록 가능하게 한다. ( 아래 그림은 이해하기 쉽게 표현된 그림 https://rise.cs.berkeley.edu/blog/ray-tips-for-first-time-users/ )
    - 코드를 보게 되면 _, futures = ray.wait(futures)로 되어있는데 futures는 남아있는 작업을 의미한다.
    - _ 는 현재 작업한 작업 결과물이기 때문에 이를 활용하여 계속하여 반영시켜 주면 그때그때 반영시킬 수 있다.
- ray.shutdown 프로세스 종료
    
    ![](https://docs.google.com/drawings/d/sI6-tbAQQ032tvCU7vqQisw/image?w=518&h=450&rev=818&ac=1&parent=1Rv_NVdclEuAup4zdhTxtR8huawu7lX4ucds3sSiQlpQ)
    

```bash
import ray
from datetime import datetime
from random import random
import psutil

@ray.remote
def task(lst, x):
    return lst[x]*10

def memory_usage(message: str = 'debug'):
    p = psutil.Process()
    rss = p.memory_info().rss
    rss = rss / 2**20
    print(f'[{message}] memory usage : {rss:10.5f} MB')

if __name__ == "__main__":
    memory_usage('#1')
    cnt = 10
    start_time = datetime.now()
    lst = [random() for _ in range(cnt)]
    number_lst = ray.put(lst)
    result = ray.get([task.remote(number_lst, idx) for idx in range(cnt)])
    end_time = datetime.now()
    print(f'result : {end_time - start_time}')
    memory_usage('#2')
    print(result)
    ray.shutdown()
```

<a href="https://ibb.co/92Q71f3"><img src="https://i.ibb.co/1KFkPpJ/Untitled-3.png" alt="Untitled-3" border="0"></a><br /><a target='_blank' href='https://usefulwebtool.com/convert-text-to-binary'></a><br />

```bash
import ray
from datetime import datetime
from random import random
import psutil

@ray.remote
def task(lst, x):
    return lst[x]*10

def memory_usage(message: str = 'debug'):
    p = psutil.Process()
    rss = p.memory_info().rss
    rss = rss / 2**20
    print(f'[{message}] memory usage : {rss:10.5f} MB')

if __name__ == "__main__":
    memory_usage('#1')
    cnt = 10
    start_time = datetime.now()
    lst = [random() for _ in range(cnt)]
    number_lst = ray.put(lst)
    futures = [task.remote(number_lst, idx) for idx in range(cnt)]
    while len(futures):
        object_id, futures = ray.wait(futures)
        result = ray.get(object_id[0])
        print(result)
    end_time = datetime.now()
    print(f'result : {end_time - start_time}')
    memory_usage('#2')
    ray.shutdown()
```

<a href="https://ibb.co/ZJnjQKx"><img src="https://i.ibb.co/sP0fTQq/Untitled-2.png" alt="Untitled-2" border="0"></a><br /><a target='_blank' href='https://usefulwebtool.com/convert-text-to-binary'></a><br />

##### 성능 비교
---
- 병렬 처리를 왜하는가? 기존의 연산 속도를 나누어서 더 빠르게 하기 위함인데 엄청나게 많은 양의 데이터가 아니고서는 단일 프로세스에서 동작하는게 더 빠르다.
- 그래서 테스트하여 보았다..
- 결과는 왜 Ray 속도가 엄청 느리다. -> 아무래도 클러스터 환경을 구성하고 엄청나게 많은 양의 데이터면 빠를 수도 있다.. ( 토스 같은 곳에서는 데이터를 하둡 적재하는데만 하루정도를 쓴다고 하니 )
- 개념을 완벽하게 잡지 못해서인걸까.. 코드를 작성하고 구현하는 부분은 확실히 간단한거 같으나, 성능 면에서는 물음표이다. 내가 한개의 로컬 8코어 CPU를 사용해서 인걸까.. 피클링 없이 인메모리 처리하면 오버헤드가 그만큼 없어져야 하는데 이건 뭐.. 성능이 박살난 정도이다.
- 나중에 기회가 대면 다시 써볼 예정이나, 현재로써는 왜 쓰는지 모르겠다.

```bash
from random import random
from ray.util.multiprocessing import Pool as RayPool
from multiprocessing import Pool as MultiPool
import time

cpu_nums = 8

def task(x):
    return x*10

def ray_map_test(lst):
    return raypool.map(task, lst)

def mp_map_test(mppool, lst):
    result = mppool.map(task, lst)
    mppool.close()
    return result

def sequentail_map_test(lst):
    return [task(i) for i in lst]

def sequentail_map_test2(lst):
    return list(map(task, lst))

def generate_random_data(number):
    return [random() for _ in range(number)]

if __name__ == "__main__":
    mppool = MultiPool(processes=cpu_nums)
    raypool = RayPool(processes=cpu_nums)
    # generate random data
    number = 100
    random_list = generate_random_data(number)
    print(f'generate number : {number}')

    # Ray
    s = time.time()
    result = ray_map_test(random_list)
    print(f'Ray Parallel time : {str(time.time() - s )}')

    # MultiProcessing
    s = time.time()
    result = mp_map_test(mppool, random_list)
    print(f'MP Parallel time : {str(time.time() - s)}')

    s = time.time()
    result = sequentail_map_test(random_list)
    print(f'Sequential time : {str(time.time() - s )}')

    s = time.time()
    result = sequentail_map_test2(random_list)
    print(f'Sequential time2 : {str(time.time() - s)}')
```

<a href="https://ibb.co/qn7tsk6"><img src="https://i.ibb.co/b3QSBzt/Untitled-4.png" alt="Untitled-4" border="0"></a>
<a href="https://ibb.co/9Gck7fm"><img src="https://i.ibb.co/3k49n85/Untitled-3.png" alt="Untitled-3" border="0"></a>
<a href="https://ibb.co/x36F0Pw"><img src="https://i.ibb.co/yVq41GK/Untitled-5.png" alt="Untitled-5" border="0"></a><br /><a target='_blank' href='https://usefulwebtool.com/convert-text-to-binary'></a><br />

참조 

- https://rise.cs.berkeley.edu/blog/ray-tips-for-first-time-users/
- [https://jost-do-it.tistory.com/entry/Python-Ray-라이브러리를-이용한-코드-병렬-처리와-이에-대한-고찰](https://jost-do-it.tistory.com/entry/Python-Ray-%EB%9D%BC%EC%9D%B4%EB%B8%8C%EB%9F%AC%EB%A6%AC%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EC%BD%94%EB%93%9C-%EB%B3%91%EB%A0%AC-%EC%B2%98%EB%A6%AC%EC%99%80-%EC%9D%B4%EC%97%90-%EB%8C%80%ED%95%9C-%EA%B3%A0%EC%B0%B0)
- https://stackoverflow.com/questions/64247663/how-to-use-python-ray-to-parallelise-over-a-large-list
- https://zzsza.github.io/mlops/2021/01/03/python-ray/