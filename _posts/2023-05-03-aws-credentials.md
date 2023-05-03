---
layout: post
title:  "AWS Credentials"
author: Kimuksung
categories: [ AWS, IAM ]
tags: [ AWS, Credentials]
image: assets/images/aws-iam.svg
# 댓글 기능
comments: False
---

##### AWS Credentials 발급

---

1. Open IAM Console
2. 메뉴에서 User를 선택
3. IAM User 선택합니다.
4. **Security credentials(보안 자격 증명)** 탭을 연 다음 **Create access key(액세스 키 생성)**를 선택합니다.
5. 새 액세스 키를 보려면 [**Show**]를 선택합니다. 자격 증명은 다음과 같을 것입니다.
    - 액세스 키 ID: `access_key_id`
    - 보안 액세스 키: `access_key`
6. 키 페어 파일 다운로드 [**Download .csv file**] 안전한 위치에 키와 함께 .csv 파일을 저장합니다.

<br>
가이드 : [링크](https://docs.aws.amazon.com/ko_kr/powershell/latest/userguide/pstools-appendix-sign-up.html)