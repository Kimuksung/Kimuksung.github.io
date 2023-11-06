---
layout: post
title:  "Redshift S3 데이터 업로드"
author: Kimuksung
categories: [ Redshift ]
tags: [ Redshift ]
image: assets/images/redshift.png
# 댓글 기능
comments: False
---

Redshift Copy 명령어를 통해 S3 파일 정보 업로드하기

##### 완료 코드

```sql
COPY table_name (col_name1, col_name2)
FROM 's3://bucket/file_dir/file_name.csv'
CREDENTIALS 'aws_access_key_id=key_id;aws_secret_access_key=key'
ACCEPTINVCHARS delimiter ','
FORMAT AS CSV
IGNOREHEADER 1;
```

##### 오류 확인 시

- 에러 문구는 `ERROR: permission denied for relation stl_load_errors` 발생
- https://stackoverflow.com/questions/72243390/permission-denied-for-relation-stl-load-errors-on-redshift-serverless 를 참고하여 확인

```sql
select *
from sys_load_error_detail
order by start_time desc;
```

##### 문제 접근 방법 
- 에러 코드 → 접근 방법

1. String contains invalid or unsupported UTF8 codepoints. Bad UTF8 hex sequence: 84 (error 3)
    1. utf8을 허용하게 하기 위해서 `ACCEPTINVCHARS` 추가

1. Delimiter not found
    1. `delimiter ','` 추가
    
2. Multibyte character not supported for CHAR (Hint: try using VARCHAR). Invalid char: 84
    1. varchar type을 char type으로 변경
    
3. Delimiter not found
    1. 다시 확인하여 보니 xlsx 확장자로 구성이 되어있어 csv 파일로 변경