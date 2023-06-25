---
layout: post
title:  "Spark Csv -> RDD"
author: Kimuksung
categories: [ Spark ]
#blog post image
image: assets/images/emr_spark.png
comments: False
---

Spark 기본 연동도 마쳤으니 RDD와 Dataframe을 상세히 써보려고 합니다.

제일 기초가 되는 Csv 파일로 부터 데이터를 읽어 Map Reduce 과정을 거쳐보려고 합니다.

간단하게 id, 생성 시간, 업데이트 시간, 삭제 시간, 결제 금액, 결제 결과, 카드 id, 사용자 id와 같은 데이터가 있다고 구성하였습니다.

원하는 결과는 각 월별로 실제 승인된 결제 금액이 얼마인지를 뽑아내려고 합니다.

##### 1. CSV 파일 읽기
---
- SparkContext의 textfile을 사용합니다.
- 헤더 유무에 따라 파일을 읽어야 하는 구조가 달라질 수 있습니다.
    - header = `rdd.first()`
    
    ```bash
    from pyspark.sql import SparkSession
    
    file_dir = "datas/"
    file_name = "payment-test.csv"
    
    spark = SparkSession.\
            builder.\
            appName("test").\
            master("spark://spark-master:7077").\
            config("spark.jars", "jars/mysql-connector-java-8.0.27.jar").\
            getOrCreate()
    
    sc = spark.sparkContext
    
    rdd = sc.textFile(file_dir+file_name)
    
    header = rdd.first()
    data_rdd = rdd.filter(lambda x: x!= header)
    ```
    

##### 2. Mapping
---
- Spark의 핵심이 되는 Map-Reduce 중 하나로 데이터를 Mapping 할 수 있습니다.
- 한줄 씩 읽은 뒤 `,` 로 칼럼을 나누어줍니다.
- CSV 파일에서도 모든 칼럼이 아닌 특정 칼럼만 가져오고 싶은 경우에는 미리 인덱스를 빼주어 접근하도록 구성하였습니다.
    
    ```bash
    file_dir = "datas/"
    file_name = "payment-test.csv"
    column_list = ['id', 'created_at', 'money_paid']
    
    rdd = sc.textFile(file_dir+file_name)
    
    header = rdd.first()
    data_rdd = rdd.filter(lambda x: x!= header)
    
    # column_name -> index 접근
    header_list = header.split(",")
    index_list = [header_list.index(column_name) for column_name in column_list]
    
    # mapping
    select_rdd = data_rdd.map(
        lambda x:tuple(x.split(",")[index] for index in index_list))
    select_rdd.take(5)
    ```
    
- 추가적으로 칼럼별 타입 변경 처리입니다.
- 비어있는 값에 대해 예외처리를 해주었습니다.
- 문제점이 Json 데이터가 들어있는 경우 `,`를 기준으로 나누기 때문에 Json 내부 데이터가 각 칼럼으로 파싱된다는 점입니다.
- dataframe을 활용하여 shcema를 지정하고 값을 부르게 되면 더 편하게 사용 가능하기 때문에 이부분은 넘어가겠습니다.
    
    ```bash
    rdd = sc.textFile(file_dir + file_name)
    header = rdd.first()
    data_rdd = rdd.filter(lambda x: x != header)
    
    # 데이터 타입 매핑
    data_type_map = {
        'id': str,
        'result': str,
        'money_paid': int,
        'created_at': lambda x: to_timestamp(x, 'yyyy-MM-dd HH:mm:ss')
    }
    
    def process_empty_value(value, data_type):
        if value.strip() == '':
            return None
        try:
            return data_type(value)
        except:
            return value
    
    # 데이터 타입 변환 및 출력
    select_rdd = data_rdd.map(lambda x: tuple(
        process_empty_value(x.split(",")[index], data_type_map.get(column_name))
        for index, column_name in enumerate(header.split(","))
    ))
    
    select_rdd.take(5)
    ```
    

##### 3. Reduce
---
- RDD에서는 아래 두 함수를 이용하여 원하는 결과값을 도출합니다.
- `reducebykey` = mapping한 데이터를 원하는 결과로 묶어줍니다.
- `sortbykey` = 원하는 Key 값을 기준으로 정렬
    
    ```python
    # CSV -> RDD
    # RDD를 활용한 reducebykey-sortbykey
    from pyspark.rdd import RDD
    from operator import add
    
    sc = spark.sparkContext
    file_dir = "datas/"
    file_name = "payment-test.csv"
    
    data_map = {
        'created_at' : lambda x: x[:7],
        'money_paid' : int
    }
    
    column_list = ['created_at', 'money_paid']
    rdd = sc.textFile(file_dir+file_name)
    header = rdd.first()
    
    data_rdd = rdd.filter(lambda x: x!= header)
    
    # column_name -> index 접근
    header_list = header.split(",")
    index_list = [header_list.index(column_name) for column_name in column_list]
    
    map_rdd = data_rdd.map(
        lambda x:tuple( data_map[column_name](x.split(",")[index]) for index, column_name in zip(index_list, column_list))
    )
    
    print( f'map_rdd : {isinstance(map_rdd, RDD)}')
    reduce_rdd = map_rdd.reduceByKey(lambda a,b: a+b)
    print( f'reduce_rdd : {isinstance(reduce_rdd, RDD)}')
    sortby_rdd = reduce_rdd.sortByKey(True, 1)
    print( f'sortby_rdd : {isinstance(sortby_rdd, RDD)}')
    
    sortby_rdd.collect()
    >
    map_rdd : True
    reduce_rdd : True
    sortby_rdd : True
    [('2022-08', 8064000),
     ('2022-09', 9135000),
     ('2022-10', 7014000),
     ('2022-11', 8267000),
     ('2022-12', 8449000),
     ('2023-01', 8856000)]
    ```