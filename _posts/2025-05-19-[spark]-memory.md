---
layout: post
title:  "Spark Memory 동작 원리와 AWS EMR Serverless에 적용기"
author: Kimuksung
categories: [ Spark ]
#blog post image
image: assets/images/emr_spark.png
comments: False
featured: True
---

<br>
Spark Memory 구조에 대해 알아보겠습니다.

Spark는 JVM 기반으로 동작합니다.

Pyspark를 사용하는 경우 Python Procesr가 존재하나
기본적으로 Driver Process + Executor process 구조는 동일합니다.

Spark의 실행 모드에 따라 동작 방식을 살펴보고, 실제 AWS Emr serverless 환경에서 사용해보면서 설정 해보는 과정을 적어보려고 합니다.

그렇다면 JVM부터 하나씩 알아가보겠습니다.

<br>

###### JVM(Java Virtual Machine)

---

- java는 기본적으로 java를 컴파일한 class 파일을 가지고 Class Loader가 Runtime Data Area에 할당 후 실행 시킴
- 상세 내용은 기본적인 것이니 생략

<table>
<tr>
<td><img src="https://i.imgur.com/sQtRMJi.png" width="400"/></td>
<td><img src="https://i.imgur.com/jy7hUPP.png" width="400"/></td>
</tr>
</table>

###### Runtime Area

---

- Runtime Area는 OS로 부터 할당 받은 메모리 공간
- 일반적인 메모리 - Stack, Heap, Code, Static

1. PC Register 
2. JVM Stack <mark>Stack</mark> 
3. Native Method Stack <mark>Stack</mark> 
4. Method Area <mark>Static</mark> + <mark>Code</mark> 
5. Heap <mark>Heap</mark>

<table>
<tr>
<td><img src="https://i.imgur.com/wRumeNd.png" width="400"/></td>
<td><img src="https://i.imgur.com/UmB4MwS.png" width="400"/></td>
</tr>
</table>

<br>

###### 모드 별 Driver, Executor

---

- Local Mode에서는 1JVM = Driver + Executor
- Client Mode에서는 Client JVM(Drvier) + Cluster(N Executor)
- Cluster Mode에서는 Job을 Submit하여 처리하는 Client JVM과 Cluster(Driver + N Executor) 구조

<table>
<tr>
<th>Local Mode</th>
<th>Client Mode</th>
<th>Cluster Mode</th>
</tr>
<tr>
<td><img src="https://i.imgur.com/dnc86x4.png" width="400"/></td>
<td><img src="https://i.imgur.com/UABWsNM.png" width="400"/></td>
<td><img src="https://i.imgur.com/D0uNiCW.png" width="400"/></td>
</tr>
</table>

<br>

##### Spark Memory (Unified Memory)
---
- 과거에는 Static Memory Manager가 사용되었지만, 현재는 Execution Memory 와 Storage Memory 경계선을 자유롭게 넘나들 수 있는 Unified Memory Manager 에 대해서만 설명
- JVM에서도 On-Heap Memory Managment (In-Memory), Off-Heap Memory Managment (External-Memory) 가 존재

![spark memory](https://i.imgur.com/X5XW6mw.png) 

###### 온힙 메모리 (On-Heap Memory)
- 온힙 메모리의 크기는 Spark 애플리케이션이 시작될 때 --executor -memory 또는 spark.executor.memory 매개변수 값

###### Reserved Memory
- 시스템용 예약된 메모리로 300MB 고정
- 격리되어 있는 메모리(접근 불가)
- 제어 정보(metadata) 저장용 메모리

###### User Memory
- 사용자 정의 데이터 구조, Spark 내부 메타데이터, 사용자가 생성한 모든 UDF, RDD 종속성 정보에 대한 정보 등 RDD 변환 작업에 필요한 데이터를 저장하는 데 사용되는 메모리

###### Spark Memory
Execution Memory 와 Storage Memory 메모리의 경계선은 언제든지 넘나들수 있게끔 설계
1. **Execution Memory**
- Execution 메모리는 Spark 작업을 실행하는 동안 필요한 개체를 저장
- E.g) ShuffleMemoryManager, TaskMemoryManager, ExecutorMemoryManager

2. Storage Memory
- 모든 캐시된 데이터, 브로드캐스트 변수 및 언롤 데이터 등을 저장
- LRU(Least Recent Used) 메커니즘을 기반으로 오래된 캐시된 개체를 제거하여 새 캐시 요청을 위한 공간을 지웁니다.

<table>
<tr>
<td><img src="https://i.imgur.com/KVBFYPJ.png" width="400"/></td>
<td><img src="https://i.imgur.com/kPinx5C.png" width="400"/></td>
</tr>
</table>

<br>

---

이제 실제 위에 배운 내용 처럼 동작하는지 알아봅시다.

EMR Serverless를 사용중이고 driver와 executor는 config 값을 주어 설정해야합니다.

아래 표를 보시면 이상한 점을 발견할 수 있습니다.

<table>
<th>1 driver + 10 executor</th>
<th>1 driver + 1 executor</th>
<tr>
<td><img src="https://i.imgur.com/VUZ2LLm.png" width="400"/></td>
<td><img src="https://i.imgur.com/evZmApf.png" width="400"/></td>
</tr>
</table>

<br>

- case1 : driver와 executor의 메모리를 4GB로 설정했음에도 실제 메모리는 2GB임을 볼 수 있구요
- case2 : 1 driver + 10 executor 로 설정을 해주었는데 executor가 5밖에 뜨지 않았네요
- case3 : 1 driver + 1 executor로 설정을 해주었는데 executor가 3개나 떠 있습니다.

case1) 위에서 본것처럼 실제 동작하는 storage memory 영역은 4GB - 300MB를 제외하고 남은 3.7GB
이중 3.7 * 0.6 = 2.22 로 관리되고 Unified Memory Pool 로 공통적으로 사용되어 2GB라고 나오는게 
아닌가하고 추정합니다.

case2) Application 설정 값에 최대 Disk 150GB까지 설정 할 수 있도록 설정했습니다. executor의 disk 값은 20GB로 5개까지 provision 후
나머지 provision하기에는 용량이 작은 문제였습니다.

```
com.amazonaws.emr.serverless.shaded.software.amazon.awssdk.services.emrserverlessresourcemanager.model.ApplicationMaxCapacityExceededException: Worker could not be allocated as the application has exceeded maximumCapacity settings: [disk: 150 GB]
```

<img src="https://i.imgur.com/1HrhYGv.png" width="400"/>

case3) aws emr serverless 환경에서는 dynamic allocation 값이 할당되는데, 별도의 값을 지정하지 않는다면
spark.dynamicAllocation.enabled = true 세팅 값으로 최소 executor 값이 설정됩니다. 이에 따라 AWS Emr serverless 정책상 최소 3개의 executor가 필요합니다.
실제 값을 보면 spark.dynamicAllocation.initialExecutors, spark.dynamicAllocation.minExecutors = 3 으로 설정된것을 볼 수 있습니다.

```
%%configure -f
{
    "driverMemory":"4G",
    "driverCores":1,
    "executorMemory":"4G",
    "executorCores":2,
    "numExecutors":10,
}
```


##### 참조

---

- https://spidyweb.tistory.com/514
- https://velog.io/@bbkyoo/Spark-Memory-%EC%A0%95%EB%A6%AC
- https://velog.io/@busybean3/Apache-Spark-%EC%95%84%ED%8C%8C%EC%B9%98-%EC%8A%A4%ED%8C%8C%ED%81%AC%EC%9D%98-%EB%A9%94%EB%AA%A8%EB%A6%AC-%EA%B4%80%EB%A6%AC%EC%97%90-%EB%8C%80%ED%95%B4%EC%84%9C
- https://d2.naver.com/helloworld/1329
- https://d2.naver.com/helloworld/1230
- https://lucas-owner.tistory.com/38 

