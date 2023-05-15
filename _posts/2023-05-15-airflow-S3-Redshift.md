---
layout: post
title:  "Airflow를 활용하여 S3 Parquet 데이터 Redshift 적재"
author: Kimuksung
categories: [ Airflow, S3, Redshift ]
tags: [ Airflow, S3, Redshift ]
image: assets/images/airflow.png
# 댓글 기능
comments: False
---

안녕하세요  
오늘은 S3 Parquet 데이터를 Redshift에 적재하는 방법에 대해 공유드리려고 합니다.

<br>

##### S3 to Redshift
---
- `MWAA IP는 고정` ( 2 Public Subnet )
- Public Subnet이 2개임으로 Redshift Inbound에 Ip 2개다 전부 추가해주어야 한다.
- S3에는 Value값만 들어가 있다.
- Parquet type으로 데이터 전달.

##### Step1) create table redshift
- Table 구성이 먼저 되어있어야 값이 추가 가능합니다.

##### Step2) S3 → Redshift
- Parquet File 업로드
- **`Copy`** 명령어 지원 ( [참고](https://docs.aws.amazon.com/ko_kr/redshift/latest/dg/tutorial-loading-run-copy.html) )

```python
# Result Copy Command
# 문자로 된 파일을 ,로 구분하여 데이터 입력
copy table 
from 's3://{bucketname}/{key}'
credentials 'aws_access_key_id=id;aws_secret_access_key=key'
copy option
; 
```
- redshift_host
- aws_access_key,id
- **s3_bucket(required)** = Bucket 이름
- **s3_key(required)** = File Directory
- **schema (required)**
- collection**(required)**
- copy_options

```python
def copy_redshift(
    aws_access_key_id,
    aws_secret_access_key,
    schema,
    collection,
    file_name,
    s3_prefix,
		redshift_host,
		**s3_bucket**
):
    import redshift_connector
    aws_info = {
            "region_name": "ap-northeast-2",
            "aws_access_key_id": aws_access_key_id,
            "aws_secret_access_key": aws_secret_access_key
        }
    bucket_name=**s3_bucket**
    s3_prefix=s3_prefix
    copy_sql = f"""
	    COPY {schema}.{collection}
	        FROM 's3://{bucket_name}/{s3_prefix}{file_name}'
	        credentials
	        'aws_access_key_id={aws_info['aws_access_key_id']};aws_secret_access_key={aws_info['aws_secret_access_key']}'
	        format as parquet;
    """.format()

    conn = redshift_connector.connect(
            host= redshift_host,
            database='',
            user= '',
            password= '!',
            auto_create= True
        )
    cursor: redshift_connector.Cursor = conn.cursor()

    print(copy_sql)
    cursor.execute( copy_sql )
    conn.commit()
```

##### 참고 자료

- [https://airflow.apache.org/docs/apache-airflow-providers-amazon/stable/operators/transfer/s3_to_redshift.html](https://airflow.apache.org/docs/apache-airflow-providers-amazon/stable/operators/transfer/s3_to_redshift.html)
- [https://airflow.apache.org/docs/apache-airflow-providers-amazon/1.0.0/operators/s3_to_redshift.html](https://airflow.apache.org/docs/apache-airflow-providers-amazon/1.0.0/operators/s3_to_redshift.html)
- [https://tkwon.tistory.com/6](https://tkwon.tistory.com/6)