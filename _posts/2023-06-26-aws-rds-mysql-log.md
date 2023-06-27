---
layout: post
title:  "AWS RDS Mysql Log 데이터 만들기"
author: Kimuksung
categories: [ RDS ]
tags: [ RDS, AWS, Mysql ]
# 댓글 기능
comments: False
image: "https://i.ibb.co/wYVBLFY/rds.png"
---

##### Enable Mysql Query Log
---
https://stackoverflow.com/questions/6479107/how-to-enable-mysql-query-log

- mysql 버전이 **5.1.29 미만**인 경우
    - /etc/my.cnf → [mysqld] section
        
        ```sql
        # add etc/my.cnf
        log = /path/to/query.log
        ```
        
        ```sql
        SET general_log = 1;
        ```
        
- mysql **5.1.29이상**
    - `log` option is deprecated
    - my.cnf → [mysqld] section에 추가
        
        ```sql
        general_log_file = /path/to/query.log
        general_log      = 1
        ```
        
        ```sql
        SET global general_log = 1;
        ```
        

##### AWS RDS Parameter Group 설정
---
- 실제 RDS MYSQL Log 설정

1.Parameter 설정값 설정
- AWS RDS → Parameter Groups → Create Parameter group or Edit
- Genral log
    - genral log는 실시간, 대규모 환경에서는 사용하지 않는다. ( 일반적으로 데이터가 많다면 Mysql을 안쓰지 않나..? )
    - general_log = 1
    
    ![](https://i.ibb.co/YBcT7xY/2023-06-26-3-20-40.png)
    
- slow query log
    - 쿼리가 특정 이상 시간(long_query_time)이라면 로그를 별도로 저장
    - slow_query_log = 1
    - long_query_time = 0 (쿼리 테스트를 위하여 설정한 값)
    - log_output = Table (file로 하면 log가 보인다던데 적용이 되지 않아 Table로 처리)
    
    ![](https://i.ibb.co/dLYS23R/2023-06-26-3-21-23.png)
    ![](https://i.ibb.co/85JMxNk/2023-06-26-3-22-07.png)
    ![](https://i.ibb.co/fNPNpsp/2023-06-26-6-26-38.png)
    

기타 - [참조](https://sangchul.kr/entry/aws-Amazon-RDS-Aurora-Serverless-%EB%A1%9C%EA%B9%85-%EC%84%A4%EC%A0%95%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95)

| Aurora MySQL | 설명 |
| --- | --- |
| general_log | 일반 로그를 생성합니다. 활성화하려면 1로 설정합니다. 기본값은 비활성화(0)입니다. |
| **```log_queries_not_using_indexes```** | 인덱스를 사용하지 않는 느린 쿼리 로그에 모든 쿼리를 로깅합니다. 기본값은 비활성화(0)입니다. 이 로그를 활성화하려면 1로 설정합니다. |
| long_query_time | 빠른 실행 쿼리가 느린 쿼리 로그에 로깅되지 않도록 합니다. 0에서 31536000 사이의 부동 소수점 값으로 설정할 수 있습니다. 기본값은 0(비활성화)입니다. 10초 |
| server_audit_events | 로그에 캡처할 이벤트 목록입니다. 지원되는 값은 CONNECT, QUERY, QUERY_DCL, QUERY_DDL, QUERY_DML 및 TABLE입니다. |
| server_audit_logging | 서버 감사 로깅을 활성화하려면 1로 설정합니다. 이 옵션을 설정하면 server_audit_events 매개 변수에 감사 이벤트를 CloudWatch 나열하여 보낼 감사 이벤트를 지정할 수 있습니다. |
| slow_query_log | 느린 쿼리 로그를 생성합니다. 느린 쿼리 로그를 활성화하려면 1로 설정합니다. 기본값은 비활성화(0)입니다. |

<br>

2.RDS Instance Reboot

- Pending reboot 상태가 되면 RDS Instance Reboot
    
    ![](https://i.ibb.co/d7qmk9m/2023-06-26-6-33-20.png)
    

<br>

##### Enable AWS RDS Mysql Query Log to CloudWatch
---
- 주의점 : **이 부분은 로그가 생성 된 이후 CloudWatch로 보내는 옵션이지 로그를 설정하는 옵션이 아니다.**
- AWS RDS → Database → Modify → configuration
- **Reader Instance**는 안보이는거 같다.
1. Open the Amazon RDS console at https://console.aws.amazon.com/rds/.
2. In the navigation pane, choose **Databases**.
3. Choose the Aurora MySQL DB cluster that you want to publish the log data for.
4. Choose **Modify**.
5. In the **Log exports** section, choose the logs that you want to start publishing to CloudWatch Logs.
6. Choose **Continue**, and then choose **Modify DB Cluster** on the summary page.
- https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_LogAccess.html
- https://repost.aws/ko/knowledge-center/rds-aurora-mysql-logs-cloudwatch
- https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Integrating.CloudWatch.html

![](https://i.ibb.co/1dqkrg8/2023-06-26-1-57-27.png)

##### AWS RDS LOG 조회하기
---
- AWS RDS → Database → Log&Event → Logs → View
- https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_LogAccess.Procedural.Viewing.html
- https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_LogAccess.Procedural.Downloading.html

##### SQL Connection Log, Slow Query 조회
---
- 실제 연결 로그
- General Log 값이 세팅되어있는 경우만 추적 가능
    
    ```sql
    # 연결 정보
    SELECT
        CONVERT_TZ(event_time,'UTC','Asia/Seoul') as connect_time,
        user_host,
        argument
    FROM mysql.general_log
    WHERE command_type = 'Connect' limit 10;
    
    # general log
    select 
        CONVERT_TZ(event_time,'UTC','Asia/Seoul') as ktc_time,
        user_host,
        command_type,
        argument
    from mysql.general_log
    order by event_time;
    
    # slow query log
    SELECT
        CONVERT_TZ(start_time,'UTC','Asia/Seoul') as event_time,
        user_host,
        query_time,
        lock_time,
        rows_sent,
        rows_examined,
        db,
        sql_text
    FROM mysql.slow_log
    order by event_time;
    ```
    

###### 참조
---
- https://manvscloud.com/?p=1072
- [https://sangchul.kr/entry/aws-Amazon-RDS-Aurora-Serverless-로깅-설정하는-방법](https://sangchul.kr/entry/aws-Amazon-RDS-Aurora-Serverless-%EB%A1%9C%EA%B9%85-%EC%84%A4%EC%A0%95%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95)