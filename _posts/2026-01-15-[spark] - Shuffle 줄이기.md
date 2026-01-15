---
layout: post
title:  "EMR Shuffle 줄여 속도 개선하기"
author: Kimuksung
categories: [Spark]
tags: [Spark, Shuffle]
image: assets/images/spark.png
comments: True
featured: True
---

<br>

안녕하세요    
오늘은 실무에서의 EMR 성능 최적화 인사이트를 공유 드리려고 합니다.    
오늘의 목표 : EMR Shuffle 줄이기입니다.

<br>

##### 결론 : EMR Shuffle 줄이기
> 기준이 되는 Column으로 연산을 반복 수행한다면 Dataframe.repartition() 후 Transformation의 추가적인 데이터의 이동을 줄일 수 있습니다. 

- Shuffle은 디스크 I/O와 네트워크 오버헤드를 동반하는 가장 무거운 작업입니다.
- 데이터 처리 시 사용자 ID, 상품 ID, 매장 ID 등 특정 데이터를 기준으로 Group By, Window Function 등을 수행하는 경우가 많습니다.
- shuffle로 인한 잦은 데이터의 이동이 발생으로 리소스를 사용하고 시간이 오래걸린다는 점을 인지했습니다.
- 반복되는 RDD에 대해 공통적으로 재사용 할 수 없을까? 라는 방식을 통해 접근하였습니다.
- 56s 걸리던 작업을, repartition 한 번(34s) + 이후 연산들(1.3s) 으로 37.5% 감소 했습니다.
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2Fm0bCt%2FdJMcachxAhj%2FAAAAAAAAAAAAAAAAAAAAACXBbPSUXKaXfDsc9Ea7hme3PYThO5uYqnyTPZMSyp0m%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1769871599%26allow_ip%3D%26allow_referer%3D%26signature%3Dbr7uvn28W3vtcivDmew6gT5ab5U%253D" />
  <figcaption style="text-align: center;"> EMR Shuffle 결과 </figcaption>


<br>

##### Spark Partition이란?
---
먼저, Spark의 동작원리에 대해 이해해야합니다.
- Spark는 Optimizier를 통해 Logical -> Physical Plan -> RDD 연산을 하는 Lazy Execution 구조입니다.
- 데이터 처리 시 데이터를 잘게 쪼개는 파티션 과정을 거칩니다. Executor에게 나누어서 처리하게 위해서죠
- 파티션이 너무 작게, 크게 나누어져도 다양한 이슈들이 발생합니다. 아래 이미지와 같이 말이죠
- 또한, 가공하는 과정에서 데이터의 불균형(Skew)이 발생할 수 있습니다. filter, join, Union 과 같은 연산으로  
- 위와 같은 Wide Transformation 연산으로 인해 파티션을 재할당하는 과정은 Spark에서 가장 리소스를 많이 사용하는 영역입니다.
- 이 연산을 줄일 수 있다면, 불필요한 리소스를 사용하지 않으며 빠르게 처리가 가능하다는 점이죠
- 이 부분을 초점을 맞추어 실제 코드 개선하는 과정을 보여드리려고 합니다.
- 파티션에 따라 발생하는 문제점에 대해서는 이미지 우측에 간단하게 적어두었고 상세한 내용은 다음에 진행하겠습니다.

<div style="display: flex; justify-content: space-around; align-items: flex-start;">
<figure>
  <img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FbAcGlc%2FdJMcaiPybAp%2FAAAAAAAAAAAAAAAAAAAAAM0Q8Pm03XMqcHtdCXjrTkX8zf8uLNsCwWGtGB9p5DBN%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1769871599%26allow_ip%3D%26allow_referer%3D%26signature%3DMPzoVosIG2MtlgqsNIkJBnA3PbQ%253D" />
  <figcaption style="text-align: center;">partition 작은 경우</figcaption>
</figure>

<figure>
  <img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2Fuv76B%2FdJMcahXttGp%2FAAAAAAAAAAAAAAAAAAAAAL6gUzbMb9mEWiQW3e37VZk8fFdP5RbY3xXly9BKrY93%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1769871599%26allow_ip%3D%26allow_referer%3D%26signature%3DEq%252B8jWpbGCVouhn4%252Fxh%252BKVj3FU8%253D" />
  <figcaption style="text-align: center;">partition 큰 경우</figcaption>
</figure>
</div>
<br>

##### 원인 찾기
---
- 데이터 처리 시 사용자 ID, 상품 ID, 매장 ID 등 기준이 되는 데이터를 미리 인지합니다.
- Column 기준으로 Dataframe.repartition() 을 작업 해놓고, 이후 Transformation 에서 해당 Column 을 기준으로 연산을 수행한다면 추가적인 데이터의 이동을 줄일 수 있습니다.
- 저의 경우에는 매장의 ID를 가지고 집계를 하거나, 다른 데이터와 join 후 처리하는 과정이 필요했습니다.

```
f_event_log_df = spark.read.parquet('s3')
f_event_log_df_filtered = f_event_log_df.select('store_code', 'page_name', 'event_ts')
f_event_log_df_sum = f_event_log_df_filtered.groupBy("store_code").agg(F.count("*"))

d_store_df = spark.read.parquet('s3')
d_store_df_selected = d_store_df.select(
    F.col('store_id').alias('store_code'), 
    F.col('is_test_store')
)

df_joined = f_event_log_df_filtered.join(F.broadcast(d_store_df_selected), "store_code")
window_spec = Window.partitionBy('store_code').orderBy('event_ts')
df_final = df_joined.withColumn('rank', F.rank().over(window_spec))

# Action0
df_final.show(10)

# Action1
f_event_log_df_sum.show(5)
```

- Action0와 Action1을 살펴보면 공통된 dataframe으로 부터 shuffle(exchange)하는 과정을 거치는 것을 알 수 있습니다.
<div style="display: flex; justify-content: space-around; align-items: flex-start;">
<figure>
  <img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FcMiuw8%2FdJMcafehgsm%2FAAAAAAAAAAAAAAAAAAAAAFp6vtW0NNBr06wokyOu0qo0blLuF2ErFcvXg3Kb5q-V%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1769871599%26allow_ip%3D%26allow_referer%3D%26signature%3D6d8PSyqgZRgt%252FQk0eV4KfTJQuT0%253D" />
  <figcaption style="text-align: center;">Action0 (spark ui)</figcaption>
</figure>
  
<figure>
  <img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FbfMjAz%2FdJMcafyzmRJ%2FAAAAAAAAAAAAAAAAAAAAADsNh2EC6lSO_i9HqkhCdDWde5h1QyavU3dftIb5Y_Iq%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1769871599%26allow_ip%3D%26allow_referer%3D%26signature%3DWpmziHH7EOAoBMMqcx9oPbApkoA%253D" />
  <figcaption style="text-align: center;">Action1 (spark ui)</figcaption>
</figure>
</div>

<br>

##### 코드 개선하기
---
- 그렇다면 어떻게 개선할 수 있을까요? 공통된 영역에 대해 재사용할 수 없을까요?
- 방법은 repartition을 칼럼으로 다시 파티셔닝을 잡아둡니다. 물론, repartition이 다시 shuffling 해둔것이기에 여러 번 재사용할 때 유용합니다. 
- 파티션의 크기에 따라 기준이 되는 칼럼을 해쉬해서 파티션 크기만큼 나머지 처리 연산을 해서 진행해요.
- 최초 1회 repartition을 통해 데이터를 물리적으로 재배치하고 이를 캐싱함으로써, 이후 발생하는 여러 단계의 Wide Transformation에서 중복 Shuffle을 방지할 수 있습니다.

```
f_event_log_df_ready = f_event_log_df.select('store_code', 'screen_name', 'event_ts') \
             .repartition(200, 'store_code') \
             .cache()
# Action2         
f_event_log_df_ready.count()

f_event_log_df_ready_joined = f_event_log_df_ready.join(F.broadcast(d_store_df_selected), "store_code")
f_event_log_df_ready_ready_final = f_event_log_df_ready_joined.withColumn('rank', F.rank().over(window_spec))
# Action3
f_event_log_df_ready_ready_final.show(10)

df_sum_new = f_event_log_df_ready.groupBy("store_code").agg(F.count("*"))
# Action4
df_sum_new.show(5)
```

<div style="display: flex; justify-content: space-around; align-items: flex-start;">
<figure>
  <img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FlUTLZ%2FdJMcaiaZTr1%2FAAAAAAAAAAAAAAAAAAAAAJkocJZt0TWzB1dTlK_lyfw4C2aN_6QciCANnIZTcmuP%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1769871599%26allow_ip%3D%26allow_referer%3D%26signature%3D9HlU%252BnipTmq67CMyJ0%252FQyh7UB6A%253D" />
  <figcaption style="text-align: center;">Action3 (spark ui)</figcaption>
</figure>
  
<figure>
  <img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2F59ohq%2FdJMcaiB07qI%2FAAAAAAAAAAAAAAAAAAAAABWObh0ttxjdC1OTbZ7N_9eFOlMtF5qiAxehwr0sdt5i%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1769871599%26allow_ip%3D%26allow_referer%3D%26signature%3DqjCcAf6DNTPI8Zbq%252BCT5UbURqqM%253D" />
  <figcaption style="text-align: center;">Action4 (spark ui)</figcaption>
</figure>
</div>

---

