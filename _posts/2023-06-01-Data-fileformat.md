---
layout: post
title:  "File Format - Data Serialize"
author: Kimuksung
categories: [ File ]
tags: [ parquet, avro, orc ]
image: "https://ifh.cc/g/NG9soh.jpg"
comments: False
---

##### Parquet
---
- `Column Base`
- 저장 공간 효율(데이터 압축)
- I/O 작업 최소화
- **`병렬 처리`**, **`Vector화`**
- WORM = Write Once Read Many
- 복잡한 중첩 데이터 구조 지원
    ```python
    {
        "Users": {
            "Name": "Alice",
            "Age": 30,
            "Addresses": [
                {
                    "Street": "123 Main St",
                    "City": "Springfield",
                    "State": "IL"
                },
                {
                    "Street": "456 Maple St",
                    "City": "Hometown",
                    "State": "IL"
                }
            ]
        }
    }
    ```


##### Avro
---
- Row Base, 동적 스키마 지원, **`스키마 변경 호환성`**
- Binary format
- row = Binary / Schema = Json
- RPC(Remote Procedure Call) 지원

<br>
 
##### ORC
---
- Column 기반 Serialize
- 인코딩 스키마를 제공하여 빠른 Read,Write,처리 속도 제공
- **`Vector화`**를 지원하여 **`병렬 처리 최적화`** → 쿼리 성능 향상
- Hive에 주로 사용