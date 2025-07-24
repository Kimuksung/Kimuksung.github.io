---
layout: post
title:  "Apache Iceberg Timestamp(TimeZone) 문제 해결기 – KST와 UTC"
author: Kimuksung
categories: [ DataLake, Iceberg ]
image: assets/images/iceberg.png
comments: True
featured: True
---

Athena 기반 환경에서 Apache Iceberg를 사용할 때, Timezone과 관련된 Timestamp 값이 의도와 다르게 처리되는 문제가 발생했습니다.

특히 Spark 세션을 KST로 설정했음에도 Iceberg에 저장된 시간 값이 UTC 기준으로 해석되며, +9시간의 오차가 발생하여 분석 결과의 신뢰도에 큰 영향을 미칠 수 있는 이슈 입니다.

이 글에서는 이 문제의 원인과 해결 과정을 공유하고, 실제 적용한 방식을 전해드릴 예정입니다.

<br>

### 문제 정의

1. Iceberg의 테이블은 Athena를 사용하고 있기에, format version 2.0을 사용중입니다.
2. Apache Iceberg의 공식 스펙에 따르면, Timezone이 포함된 Timestamp는 format 3.0 부터 지원됩니다.
3. 데이터 소스는 KST Timezone 값을 제공하며, Spark로 처리한 후 Iceberg에 Append하면 값 자체는 변하지 않고 Timezone 정보가 무시된 채 UTC 기준으로 저장됩니다.
4. Spark Session Timezone을 Asia/Seoul, Etc/GMT+9로 설정했음에도 불구하고, Iceberg에서는 UTC 값 그대로 적용됩니다.

```python
spark.config("spark.sql.session.timeZone", "Etc/GMT+9")
```

### 해결 과정

카카오 기술 블로그에서(https://tech.kakao.com/posts/694) 유사한 이슈가 소개되었습니다.

- 플링크 job에서 처리 시간을 생성하면 KST 기준으로 생성되지만, 이 값을 아이스버그에 저장하면 값 자체는 변경되지 않으나 UTC 타임존의 값으로 인식됩니다. 이로 인해, 팀내 스파크 세션은 KST로 설정되어 있어 UTC로 저장된 값을 KST로 읽을 때 +9 시간이 반영되어 잘못된 시간이 반환되는 문제가 발생
- 해당 사례에서는 Timestamp를 문자열 처리하여 우회하는 방법을 선택했습니다.

하지만 상황이 조금 다릅니다.
- 내부 서비스 DB에서 Spark로 스냅샷 형태로 불러오고 있었고 파티셔닝의 기준이 되는 칼럼이 아니다.
- 총 186개의 칼럼 중 8개가 Timestamp였으며, 향후 추가 가능성이 있고 통일성 있게 처리하고자 내부 Timestamp 값은 UTC 기준으로 적재하고 있습니다.
- 컬럼마다 수동 처리하거나 문자열 변환하는 방식은 피하고자 했습니다.

<br>

### 결론
> Spark가 읽어온 컬럼이 KST 기준임을 명시적으로 알려주기 위해 다음과 같이 to_utc_timestamp() 함수를 활용했습니다

```python
F.to_utc_timestamp(F.col("col_name"), "Asia/Seoul")
```

