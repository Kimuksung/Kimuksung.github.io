---
layout: post
title:  "Process"
author: Kimuksung
categories: [ OS ]
tags: [ Process, System Call ]
# 댓글 기능
comments: False
---

안녕하세요  
오늘은 OS Process에 대해 다시 개념을 적립해보려고 합니다.



##### Process
---
- Scipt로 작성된 Program이 Loader에 의해 Main Memory에 올라가 CPU가 연산 처리되는 과정.
- CPU에 의해서 `**실행되고 있는 Program**`
- Processer가 할당하는 Object로, Dispatch가 가능한 Unit
- OS가 관리하는 최소 단위의 Job(Program)
- STACK + HEAP + DATA + CODE

Loader = 보조기억장치(Disk) → Main Memory 로 적재하는 소프트웨어   
  
<br>

##### Process 과정
---
대표적인 Status
READY
- CPU가 해당 Process를 처리 중이지 않으나, Memory에 올라가 있어 언제든 사용 가능하다.
- Context Swithching을 통해 본인의 차례가 오기를 기다리는 상태

RUN
- CPU가 해당 Process를 처리 중인 상태

BLOCK
- Process에 Event가 일어나기를 기다리는 상태
- Event가 발생하기 까지 상태 변경 불가능 ( I/O )

![https://ifh.cc/g/6tpXjx.png](https://ifh.cc/g/6tpXjx.png)

Process 생성 과정

- 부팅 시 처음 Process가 생성(PID=1)
- 처음 만들어진 Process로 부터 Fork하여 Process가 생성된다.
- 이를 구분하기 위해 PID를 가지게 된다.
- Process 생성 시 새로운 PCB가 생성

Parent Process는 2가지 행동

- Child Process가 종료될 때까지 ( Wait )
- Child Process와 함께 동작

Child Process

- Parent Process와 동일한 New Process ( Fork )
- 새로운 프로그램 실행 ( Fork → Exec )

##### System Call

---
Fork()

- Child Process는 Parent Process의 Data, Program이 `**완전 복사**`
- Multi Processing을 통해 부모,자식 프로세스 함께 동작
- Child Process는 PID 0을 반환하여 구분 가능

![https://ifh.cc/g/OSbPCv.png](https://ifh.cc/g/OSbPCv.png)

Exec()
- 새로운 Program을 실행하는 Process 대체하는 System Call
- Fork() 후에 프로그램 코드로 바꾸어 Code,Memory,File .. Resource를 변경
- Fork와 달리 Return 값이 존재하지 않는다.

![https://ifh.cc/g/RXNSDW.png](https://ifh.cc/g/RXNSDW.png)

Wait()
- Parent Process가 Child Process가 끝나기를 기다려야 하는 경우
- Parent Process가 Child Process보다 먼저 끝나게된다면, Child Process는 언제든지 종료될 수 있다.
- Child Process가 종료 → SIGCHLD → Waiting Q → Parent Process → Ready Q로 이동
- Block Mode로 Cost가 비싸다. ( OS에 의해 강제 종료 가능 )
- 자식 Process PID를 반환하여 종료 상태를 알 수 있다.

![https://ifh.cc/g/qptrAO.png](https://ifh.cc/g/qptrAO.png)

Exit()
- Process가 종료 → SIGCHLD → Parent Process → exit() 정상 종료시키는 System Call
- void exit()
- 함수 호출 후 Child Process에 할당된 Resource는 없어진다.
- Return 시에도 동일하게 처리

Kill()
- Process를 종료 시키는 System Call
- int kill(pid , sig ) 형태로 호출, sig ⇒ 보낼 시그널
    - 양의 정수 : PID를 가진 Process에게 시그널
    - 0 : 같은 Process Group에게 시그널
    - -1 : Process 전송 권한이 있는 Proccess에게 시그널
- OS는 너무 많은 Resource를 할당하거나 오래 할당하고 있으면 해당 Process를 Kill 처리  
E.g) Zombie Process, Orphan Process  
