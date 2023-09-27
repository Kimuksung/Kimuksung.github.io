---
layout: post
title:  "Python MultiProcessing"
author: Kimuksung
categories: [ Python, MultiProcessing ]
tags: [ Python, MultiProcessing ]
image: assets/images/python.png
comments: False
featured: True
---

파이썬 멀티 프로세싱 동작 원리와 어떤 상황에서 사용하는 것인지 적합한지를 직접 확인해보려고 합니다.

<br>

#### 멀티 프로세싱
---
- Process Spawning을 지원하는 패키지
- 쓰레드가 아닌 서브 프로세스를 사용 → GIL을 피하여 사용하겠다.
- *Spawning이란? linux의 fork와 유사한 형태로 자식 프로세스를 새로운 메모리에 띄워서 사용


그럼 멀티 쓰레드를 사용하지 않는것일까?

<br>

#### 멀티 쓰레드
---
- 하나의 프로세스에서 여러 쓰레드를 실행
- Context Switching으로 약간의 Delay가 존재하지만, 병렬로 처리할 수 있다.
- **파이썬에서는 I/O, Sleep과 같이 cpu 연산이 없는 경우에 좋다.**

하지만, 여기서 Python 언어의 특성으로 인한 문제가 발생한다.

- Python은 Object 형태로 관리하는데, 이 때 **Garbage Collection**과 **Reference Count**를 활용한다.
- 말 그래돌 하나의 Object 형태를 관리하기 위함인데, 여기서 인터프리터라는 오브젝트에 락을 걸어버렸다.
- GIL(글로벌 인터프리터 락) : **a mutex that protects access to Python objects, preventing multiple threads from executing Python bytecodes at once →** 하나의 쓰레드에 모든 Resource를 허락하고 이후에 Lock을 걸어 다른 쓰레드를 실행 할 수 없도록 만듭니다. **[병렬 실행 불가]**
- 결국 하나의 인터프리터에서는 하나의 쓰레드 바이트코드만 처리하는 것과 동일한데, Context Switching까지 일어나는 것이다. → **더 느리다.**
- 그러면 왜 GIL를 통해서 Lock을 관리하는가? 멀티쓰레드를 통하여 하나의 Object를 여러 쓰레드가 접근하게 된다면 Race Condition이 발생한다. [여러 쓰레드가 동시에 접근함으로, 값이 올바르지 않게 설정]

→ 동시에 실행되다보면, GC가 제대로 동작하지 않을 수 있다. → GIL을 통해 공유자원을 컨트롤 하겠다.

```bash
import random
import threading
import time

def working():
    max([random.random() for _ in range(500000000)])

# 1. single Thread
s_time = time.time()
working()
working()
e_time = time.time()
print(f'{e_time - s_time:.5f}')

# 2. multi Threads
s_time = time.time()
threads = []
for i in range(2):
    threads.append(threading.Thread(target=working))
    threads[-1].start()

[t.join() for t in threads]
    

e_time = time.time()
print(f'{e_time - s_time:.5f}')
```

<a href="https://ibb.co/5MMtDdP"><img src="https://i.ibb.co/ZLLts30/Untitled-2.png" alt="Untitled-2" border="0"></a>
<a href="https://ibb.co/WVL1Z4k"><img src="https://i.ibb.co/Sfzpb4t/image.png" alt="image" border="0"></a>

그렇다면 파이썬에서는 멀티쓰레딩을 사용하지 않는 것이 바람직한가?

- GIL은 공유 자원에 대해 발생가능한 Race Condition 문제가 발생한다는 것이다.
- **외부 연산(I/O, Sleep)**을 하느라 CPU가 아무것도 하지 않고 **기다리는 작업**을 할 때에는 MultiThread가 좋다. → 이런 상황이 많을까는 의문
- 그렇기 때문에 우리는 공유 자원을 사용하지 않는, 멀티 프로세싱으로 처리하면 더 빠르게 처리 할 수 있다. (하지만,, 메모리가 더 많이 들걸로 예상 )

<br>

#### 멀티 프로세싱
---
- 동시에 여러 프로세스를 처리하도록 수행
- 쓰레드와는 다르게, 공유 자원을 사용하지 않는다. ( 프로세스끼리 통신이 필요 )
- 이 과정에서 프로세스 fork, spawning 시 pickle, unpickle을 처리( 전체 복사 시 압축해서 메모리에 전달해주는 게 좋다고 이해)
- 일반적으로 Linux에서는 Fork로, Window와 같이 fork를 지원하지 않는 OS에서는 spawning (Function + Args) Pickle화 하여 ChildProcess에 전달

<br>

##### 멀티 프로세싱 기본 개념
---
multiprocessing 라이브러리를 활용하면 아래와 같은 알 수 있다.

- set_start_method : 멀티프로세싱 어떤 값을 세팅할 것인지 [spawn, fork, forkserver]
- get_start_method : 현재 OS에서 설정 가능한 방식
- get_context : 멀티프로세싱 설정 값
- Value : 공유 변수
- name
- ppid : 부모 프로세스 id
- pid : 프로세스 id
- daemon
- Process
- Pool

```bash
import multiprocessing as mp

# 코드 내부가 아닌 외부에서 설정을 해야한다.
# mp.set_start_method('fork')
# current_methods = mp.get_start_method()
# print(current_methods)

if __name__ == "__main__":
    print('-'*10+'main'+'-'*10)
    methods = mp.get_all_start_methods()
    cxt = mp.get_context('spawn')
    current_methods = cxt.get_start_method()
    print(current_methods)
    print(methods)

    process1 = cxt.Process()
    print(f'name : {process1.name}\ndeamon : {process1.daemon}\npid : {process1.pid}\n'
          f'alive : {process1.is_alive()}\nexitcode : {process1.exitcode}')
    process1.start()

    print(f'after pid : {process1.pid}')
```

##### 공유 변수를 사용하여 멀티 프로세싱으로 구성해보자
---
- 아래 결과 값은 어떻게 나올까?
- **결과는 1,2 둘다 나올 수 있다.**
- 이유는 공유 메모리에 저장된 값을 동시에 읽게 된다면
    - 서로 동시에 0을 읽고 +1하여 1로 반영
    - 각자 시간차가 발생하여 인지하고 1+1로 2반영
- 필요에 따라 Lock을 걸어 사용 할 수 있으나, 확인해보는 과정이기에 없이 구현

```bash
import multiprocessing as mp

def increment(shared_value):
    # 공유 메모리에 저장된 값을 1 증가시킴
    shared_value.value += 1

if __name__ == "__main__":
    # 공유 메모리로 사용할 정수를 생성
    shared_value = mp.Value('i', 0)

    # 두 개의 프로세스 생성
    process1 = mp.Process(target=increment, args=(shared_value,))
    process2 = mp.Process(target=increment, args=(shared_value,))

    # 프로세스 시작
    process1.start()
    process2.start()

    # 프로세스가 종료될 때까지 대기
    process1.join()
    process2.join()

    # 결과 출력
    print("Final shared value:", shared_value.value)
    process1.close()
    process2.close()
```

##### 멀티 프로세싱 구현
---
- Process로 구성
- 하나의 Process를 구성한 예제이다.
- 프로세스는 **Pickle, Unpickle이 처리됨을 알 수 있다.**
- getstate, setstate 함수는 pickle과 unpickle시 호출 되는 함수이다.

```bash
import multiprocessing as mp

class Test:
    def __init__(self):
        self.x = 1
        print("init")

    def __call__(self, x):
        return x + self.x

    def __getstate__(self):
        print("get state")
        return self.__dict__

    def __setstate__(self, state):
        print("set state")
        self.__dict__ = state

def custom_map(func, data):
    results = []
    for item in data:
        result = func(item)
        results.append(result)
    return results

if __name__ == "__main__":
    test = Test()
    print('-'*10)
    print(test(1))
    print('-' * 20)
    process1 = mp.Process(target=custom_map, args=(test, [1, 2, 3]))
    print('-' * 30)
    process1.start()
    print('-' * 40)
    process1.join()
    result = process1.exitcode
    print(result)
    process1.close()
```

<a href="https://ibb.co/3FdmqyJ"><img src="https://i.ibb.co/fNFd5Yc/Untitled-2.png" alt="Untitled-2" border="0"></a>

<br>

##### 부모 자식 확인
---
- parent_process와 active_children을 통하여 부모 프로세스와 자식 프로세스를 알아 볼 수 있다.
- 여기서 의문점을 가질 수 있다. Active함수가 나중에 실행되었는데 먼저 값이 호출이 되는 것인가?
- process.start() 는 말 그대로 프로세스를 시작하는 함수이며, join()이 있어야 해당 프로세스가 끝날때까지 대기한다. 그러기에 join이 없는 프로세스들을 시작하고 이후 자기 할일을 바로 시작하여 먼저 호출되는 것이다.
- active_children() 함수는 현재 살아있는 자식 프로세스를 반환, 이미 완료된 프로세스에 join하기에 주의해야한다.
- 위와 같은 과정으로 join을 먼저 실행하게 된다면, active_children 함수 값이 없게 나온다.

```bash
from multiprocessing import Process
from multiprocessing import current_process
from multiprocessing import parent_process
from multiprocessing import active_children
from time import sleep

# 멀티프로세싱 환경 설정 및 자식 프로세스
# 코드 내부가 아닌 외부에서 설정을 해야한다.
# mp.set_start_method('fork')
# current_methods = mp.get_start_method()
# print(current_methods)

def task():
    sleep(1)
    parent_pro = parent_process()
    current_pro = current_process()
    print(f'parent_process : {parent_pro.name}, {parent_pro.pid} / current_process : {current_pro.name}, {current_pro.pid}')

if __name__ == "__main__":
    process1 = Process()
    process1.name = 'MyProcess'
    process1.daemon = True
    print(f'process1 : {process1.name}, {process1.daemon}, {process1.pid}')

    current_processes = current_process()
    print(f'current_process : {current_processes.name}, {current_processes.pid}')

    process3 = Process(target=task)
    process3.start()
    process3.join()

    print('-'*10)
    processes = [Process(target=task) for _ in range(5)]
    [process.start() for process in processes]

    childs = active_children()
    print(f'Active Children Count : {len(childs)}')
    [print(child) for child in childs]

    [process.join() for process in processes]
    [process.close() for process in processes]
```

<a href="https://ibb.co/6ZYBcvb"><img src="https://i.ibb.co/pP2y6WZ/Untitled-3.png" alt="Untitled-3" border="0"></a>
<a href="https://ibb.co/ZJ4sC4H"><img src="https://i.ibb.co/26V2BV8/Untitled-2.png" alt="Untitled-2" border="0"></a>

<br>

##### 락을 활용한 멀티 프로세싱 구성
---
- cpu_count() : cpu 코어 수
- Lock 함수를 활용하여 acquire, release 하여 구성한다. (OS에서 배운 semaphore 와 동일)
- Lock이 걸려있으면 다른 프로세스에서 접근을 하지 못하기에 대기하여야 한다.
- Lock이 없다면 비슷한 시간에 모든 프로세스가 접근하는 것을 볼 수 있다.

```bash
from multiprocessing import cpu_count
from multiprocessing import current_process
from multiprocessing import Lock

# 8
num_cores = cpu_count()
print(num_cores)

# <_MainProcess name='MainProcess' parent=None started>
process = current_process()
print(process)

lock = Lock()
lock.acquire()
# acquire lock without blocking
lock.acquire(block=False)
# acquire lock with timeout
lock.acquire(timeout=10)
lock.release()
```

```bash
from multiprocessing import Lock
from multiprocessing import Process
from time import sleep
from random import random
from datetime import datetime

def task(lock, identifier, value):
    #acquire lock
    with lock:
        print(f'process {identifier} get lock and sleeping {value}s / current_time : {datetime.now()}')
        sleep(value)

def task2(identifier, value):
    print(f'process {identifier} get lock and sleeping {value}s / current_time : {datetime.now()}')
    sleep(value)

if __name__ == "__main__" :
    print('Lock multiprocessing')
    lock = Lock()
    processes = [Process(target=task, args=(lock, i, random()*10)) for i in range(10)]
    [process.start() for process in processes]
    [process.join() for process in processes]

    print('NoLock multiprocessing')
    processes = [Process(target=task2, args=(i, random()*10)) for i in range(10)]
    [process.start() for process in processes]
    [process.join() for process in processes]
```

<a href="https://ibb.co/8mZLjbJ"><img src="https://i.ibb.co/sgYdFCh/Untitled-2.png" alt="Untitled-2" border="0"></a>

<br>

##### 멀티 프로세싱으로 한번에 구현
---
- 실제로 적용해보았을 때, 메모리와 걸리는 시간이 얼마나 차이나는지를 확인하여 본다.
- 500만개와 500개를 구성하였을 때로 비교하여보았다.
- 갯수가 작으면 멀티프로세싱하기에는 컨텍스트 스위칭, 메모리 할당에 더 많은 리소스가 사용되는것으로 보인다. 고로 대용량 처리 시에만 적용하여 사용해야 할 거 같으며 코드에 따라서 직접 테스트가 필요하여 보인다.
- 클래스로 구성하여볼 수 있도록 한다.
- 테스트 중에 lambda 함수를 사용하여 구성하게 된다면, 에러가 발생하는데 이유를 찾아보니 **lambda 함수는 anonymous 함수이기 때문에** function 전달은 가능하지만, unpickle 시 name을 통하여 값을 가져올 수 없어 문제가 발생한다.

```bash
from multiprocessing import Process
from datetime import datetime
import random
import psutil

def memory_usage(message: str = 'debug'):
    p = psutil.Process()
    rss = p.memory_info().rss / 2**20
    print(f'[{message}] memory usage : {rss:10.5f} MB')

def task():
    result = max([random.random() for _ in range(5000000)])
    memory_usage("task")
    return result

if __name__ == "__main__":
    memory_usage('#1')
    print('-'*20)
    multiprocessing_start_time = datetime.now()
    processes = [Process(target=task) for _ in range(5)]
    [process.start() for process in processes]
    [process.join() for process in processes]
    memory_usage('#2')
    [process.close() for process in processes]
    multiprocessing_end_time = datetime.now()
    memory_usage('#3')
    print(f'multiprocessing time : {multiprocessing_end_time-multiprocessing_start_time}')

    memory_usage('#4')
    start_time = datetime.now()
    for _ in range(5):
        task()
    end_time = datetime.now()
    memory_usage('#5')
    print(f'normal time : {end_time-start_time}')
```

<a href="https://ibb.co/LDyrkxD"><img src="https://i.ibb.co/nqyBg7q/Untitled-3.png" alt="Untitled-3" border="0"></a>
<a href="https://ibb.co/vmN9qyT"><img src="https://i.ibb.co/bL4tmwC/Untitled-2.png" alt="Untitled-2" border="0"></a>

```bash
import multiprocessing as mp
import os

class Test:
    def __init__(self):
        self.x = 1

    def __call__(self, x):
        print(f"call pid : {os.getpid()}")
        return x + self.x

    def __getstate__(self):
        print(f"get state {os.getpid()}")
        return self.__dict__

    def __setstate__(self, state):
        print(f"set state {os.getpid()}")
        self.__dict__ = state

if __name__ == "__main__":
    print(f"current parent pid : {os.getppid()}")
    print(f"current pid : {os.getpid()}")
    print('-'*10)
    pool = mp.Pool(3)
    test = Test()
    print(pool.map(test, range(1, 5)))
    pool.close()
```

<a href="https://ibb.co/5vRfw1j"><img src="https://i.ibb.co/tcJWFH2/Untitled-4.png" alt="Untitled-4" border="0"></a>

<br>

참조 = https://it-eldorado.tistory.com/160