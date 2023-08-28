---
layout: post
title:  "error converting YAML to JSON: yaml: line: did not find expected key"
author: Kimuksung
categories: [ K8S ]
tags: [ K8S, Airflow ]
# 댓글 기능
comments: False
image: assets/images/error.png
---

kubernetes Airflow 구성 중에 helm을 통해 upgrade하는 과정에서 나타난 오류 해결 과정에 대해 설명하려고 합니다.

##### 에러 원인
**`error converting YAML to JSON: yaml: line: did not find expected key`**

#### 결론
- YAML 파일 내에서 에러가 발생한 위치의 Indent, Spacebar를 확인해본다.

```bash
$ helm upgrade --install airflow apache-airflow/airflow -n airflow -f values.yaml --debug

>
Error: failed to parse values.yaml: error converting YAML to JSON: yaml: line 2186: did not find expected key
helm.go:84: [debug] error converting YAML to JSON: yaml: line 2186: did not find expected key
```

- helm을 통하여 k8s cluster에 airflow를 구성 중에 위와 같은 에러가 발생
- 처음에는 key 값을 왜 못찾지? 라고 생각하였으나, 들여쓰기말고도 한칸더 점프해서 구성되어있는 것을 확인
- 혹시나 하여 space 된것을 지워주고 실행하였더니, 제대로 동작한다.
- line 뒤에 에러 발생 위치가 나오니 해당 부분에서 코드가 제대로 동작하게 구성되었는지 확인해본다.