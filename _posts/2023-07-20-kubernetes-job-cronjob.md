---
layout: post
title:  "kubernetes Job"
author: Kimuksung
categories: [ K8S ]
tags: [ K8S, Job ]
# 댓글 기능
comments: False
image: assets/images/k8s.png
---

#### Job
---
- 일회성으로 실행되는 서비스
- 특정 동작을 수행 뒤 종료
- Pod는 Completed가 최종 상태

<br>

##### 1. 일반적인 Job 생성
- Apiversion → batch 설정
- Deployment와 동일하게 template keyword 사용
- selector,labels은 별도로 설정안해주어도 동작
    
    ```bash
    $ kubectl apply -f default_job.yml
    $ kubectl get job
    $ kubectl describe job/hello
    ```
    
    ```bash
    # default_job.yml
    apiVersion: batch/v1
    kind: Job
    metadata:
      name: hello
    spec:
      template:
        spec:
          # Always, Never, OnFailure
          restartPolicy: Never
          containers:
          - name: hello
            image: ubuntu:focal
            command: ["sh", "-c", "echo test > test"]
    ```
    
    ![](https://i.ibb.co/JsmKCMq/2023-07-20-7-55-42.png)
    

##### 2. 여러 Job + 병렬 처리
- Airflow Task 병렬처리와 동일한 개념
- completions : 총 생성할 수
- parallellism : 동시에 실행할 수 있는 pod 수
    
    ```bash
    $ kubectl apply -f job-parallelism.yml
    $ kubectl get job
    $ kubectl describe job/hello
    ```
    
    ```bash
    # job-parallelism.yml
    apiVersion: batch/v1
    kind: Job
    metadata:
      name: hello
    spec:
      completions: 10
      parallelism: 3
      template:
        spec:
          restartPolicy: Never
          containers:
          - name: hello
            image: ubuntu:focal
            command: ["sh", "-c", "sleep 20; echo test > test]
    ```
    
    ![](https://i.ibb.co/yQmKFh3/2023-07-20-8-02-33.png)
    ![](https://i.ibb.co/hRyTJjJ/2023-07-20-8-00-06.png)
    ![](https://i.ibb.co/TKNXCXP/2023-07-20-8-01-36.png)    
    ![](https://i.ibb.co/BqWhkRF/2023-07-20-8-03-30.png)
    

##### 3. 특정 시간만 동작
---
- activeDeadlineSeconds = N초 동안만 동작
- N초가 지난다면 종료
- 아래 예시는 3초로 제한을 두고 5초 이후에 동작하도록 만든 코드
    
    ```bash
    # pod deadline 주기
    apiVersion: batch/v1
    kind: Job
    metadata:
      name: hello
    spec:
      activeDeadlineSeconds: 3
      template:
        spec:
          restartPolicy: Never
          containers:
          - name: hello
            image: ubuntu:focal
            command: ["sh", "-c", "sleep 5; echo test > test"]
    ```
    
    ![](https://i.ibb.co/pQZXT2c/2023-07-20-8-09-39.png)
    
<br>

#### CronJob
---

- kind = **`CronJob`**
- 주기적으로 특정 작업을 처리하기 위함
- successfulJobsHistoryLimit = N개 까지만 실행
- Airflow Cron 스케줄링과 동일
- Job을 생성 후 작업 수행
- Job → Pod 생성
- 일반적으로 백업, 알림 전송 등의 목적으로 사용

![](https://i.ibb.co/Vw3V0Y9/cronjob2.png)

- 매 1분마다 동작하도록 만든 job
    
    ```bash
    $ kubectl apply -f cronjob.yml
    $ kubectl get cronjob
    $ kubectl get job
    $ kubectl describe cronjob/hello-every-minute
    ```
    
    ```bash
    # cronjob.yml
    apiVersion: batch/v1
    kind: CronJob
    metadata:
      name: hello-every-minute
    spec:
      schedule: "*/1 * * * *"
      successfulJobsHistoryLimit: 5
      jobTemplate:
        spec:
          template:
            spec:
              restartPolicy: Never
              containers:
              - name: hello
                image: ubuntu:focal
                command: ["sh", "-c", "echo Hello $(date)!"]
    ```
    
    ![](https://i.ibb.co/bRPyqJk/2023-07-20-8-16-14.png)
    ![](https://i.ibb.co/xFnLmgT/2023-07-20-8-15-08.png)
    ![](https://i.ibb.co/gdMwKyS/2023-07-20-8-15-40.png)