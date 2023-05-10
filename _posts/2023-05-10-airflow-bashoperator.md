---
layout: post
title:  "Airflow BashOperator를 활용한 정보 확인"
author: Kimuksung
categories: [ Airflow ]
tags: [ Airflow, BashOperator, Linux ]
image: assets/images/airflow.png
# 댓글 기능
comments: False
---

안녕하세요

오늘은 Airflow BashOperator로 Airflow 정보를 확인하는 과정을 알아볼 예정입니다.

이유는 AWS의 기본정보를 알아 외부(Redshift,Mongodb, .. ) 와 연결하기 위함입니다.  

<br>

##### BashOperator

---

- Python Operator로 동작시키는 것이 아닌 Bash로 처리하여 코드 로직과 분리하여 구성하기 위함
- MWAA에서는 **`Read-Only FileSystem`**으로 지정되어 **`Execute 처리가 불가능`**
- 또한, jinja template으로 인하여 command 뒤에 space를 사용하여 처리해주어야 한다.
    
    ```bash
    $ chmod +x file.sh
    ```
    

##### python version

---

- 현재 Python 버전 체크

```python
# version
$ python3 --version

# airflow
BashOperator(task_id='python_version',bash_command='python --version', dag=dag)
```

##### Network

---

- Public IP 조회
    
    ```python
    $ curl ifconfig.me
    
    # dags
    BashOperator(
    	task_id='request_airflow_ip',
    	bash_command='curl ifconfig.me'
    , dag=dag)
    ```
    
- 외부 Connection 확인
    - Netcat(nc) - TCP/UDP 을 활용하여 서버 간 포트 오픈 여부 확인
    - zv option = connection 성립을 하지 않고 상태만 확인 후 끊는다.
        
        ```python
        $ nc -zv url port
        
        # dags
        BashOperator(task_id='request_redshift_connect',
                               bash_command='nc -zv url port',
                               dag=dag)
        ```
        

##### dns

---

- 서버 관련 정보를 확인하기 위해 사용
    
    ```python
    $ nslookup {url}
    
    # dags
    BashOperator(
    	task_id='request_redshift_dns',
      bash_command='nslookup {url}'
    , dag=dag
    )
    ```