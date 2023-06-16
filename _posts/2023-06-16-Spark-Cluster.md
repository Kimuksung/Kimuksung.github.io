---
layout: post
title:  "Spark Cluster 구성하기"
author: Kimuksung
categories: [ Spark ]
#blog post image
image: assets/images/emr_spark.png
comments: False
---

Docker를 활용하여 Local에 구성하며 테스트하려는 목적에 따라 구성하였습니다.

##### Spark Cluster 구성
---
- Spark Cluster = Master - 2Slave
- 전체 Cluster = 쥬피터 + Spark Cluster
- 전체 Cluster = cluster-base.Dockerfile
- SparkCluster = spark-base.Dockerfile
- Master = spark-master.Dockerfile
    - 자체 설정 값 포트 구성 Export(8080)+ Worker Node Connect Port(7077)
- Slave = spark-worker.Dockerfile
    - 자체 설정 값 포트 구성 Export(8081)
    
    ![image](https://www.kdnuggets.com/wp-content/uploads/perez-spark-docker-1.png)
    
    ```docker
    #cluster-base.Dockerfile
    ARG debian_buster_image_tag=8-jre-slim
    FROM openjdk:${debian_buster_image_tag}
    
    # -- Layer: OS + Python 3.7
    
    ARG shared_workspace=/opt/workspace
    
    RUN mkdir -p ${shared_workspace} && \
        apt-get update -y && \
        apt-get install -y python3 && \
        apt-get install -y python3-pip && \
        ln -s /usr/bin/python3 /usr/bin/python && \
        rm -rf /var/lib/apt/lists/*
    
    ENV SHARED_WORKSPACE=${shared_workspace}
    
    # -- Runtime
    
    VOLUME ${shared_workspace}
    CMD ["bash"]
    ```
    
    ```docker
    #spark-base.Dockerfile
    FROM cluster-base
    
    # -- Layer: Apache Spark
    
    ARG spark_version=3.1.1
    ARG hadoop_version=3.2
    
    RUN apt-get update -y && \
        pip3 install --upgrade pip setuptools wheel &&\
        pip3 install pandas &&\
        apt-get install -y curl && \
        curl https://archive.apache.org/dist/spark/spark-${spark_version}/spark-${spark_version}-bin-hadoop${hadoop_version}.tgz -o spark.tgz && \
        tar -xf spark.tgz && \
        mv spark-${spark_version}-bin-hadoop${hadoop_version} /usr/bin/ && \
        mkdir /usr/bin/spark-${spark_version}-bin-hadoop${hadoop_version}/logs && \
        rm spark.tgz
    
    ENV SPARK_HOME /usr/bin/spark-${spark_version}-bin-hadoop${hadoop_version}
    ENV SPARK_MASTER_HOST spark-master
    ENV SPARK_MASTER_PORT 7077
    ENV PYSPARK_PYTHON python3
    
    # -- Runtime
    
    WORKDIR ${SPARK_HOME}
    ```
    
    ```docker
    #spark-master.Dockerfile
    FROM spark-base
    
    # -- Runtime
    
    ARG spark_master_web_ui=8080
    
    EXPOSE ${spark_master_web_ui} ${SPARK_MASTER_PORT}
    CMD bin/spark-class org.apache.spark.deploy.master.Master >> logs/spark-master.out
    ```
    
    ```docker
    #spark-worker.Dockerfile
    FROM spark-base
    
    # -- Runtime
    
    ARG spark_worker_web_ui=8081
    
    EXPOSE ${spark_worker_web_ui}
    CMD bin/spark-class org.apache.spark.deploy.worker.Worker spark://${SPARK_MASTER_HOST}:${SPARK_MASTER_PORT} >> logs/spark-worker.out
    ```
    
    ```docker
    # jupyterlab.Dockerfile
    FROM cluster-base
    
    # -- Layer: JupyterLab
    
    ARG spark_version=3.1.1
    ARG jupyterlab_version=2.1.5
    
    RUN apt-get update -y && \
        apt-get install -y python3-pip && \
        pip3 install --upgrade pip setuptools wheel &&\
        pip3 install pandas &&\
        pip3 install wget pyspark==${spark_version} jupyterlab==${jupyterlab_version}
    
    # -- Runtime
    
    EXPOSE 8888
    WORKDIR ${SHARED_WORKSPACE}
    CMD jupyter lab --ip=0.0.0.0 --port=8888 --no-browser --allow-root --NotebookApp.token=
    ```
    

##### Docker-Compose 구성
---
- build.sh = image build 파일 script 구성
- docker-compose 구성 및 실행
    
    ```bash
    $ chmod +x build.sh
    $ ./build.sh
    $ docker-compose up -d
    ```
    
    ```bash
    #build.sh
    
    # -- Software Stack Version
    
    SPARK_VERSION="3.1.1"
    HADOOP_VERSION="2.7"
    JUPYTERLAB_VERSION="2.1.5"
    
    # -- Building the Images
    
    docker build \
      -f cluster-base.Dockerfile \
      -t cluster-base .
    
    docker build \
      --build-arg spark_version="${SPARK_VERSION}" \
      --build-arg hadoop_version="${HADOOP_VERSION}" \
      -f spark-base.Dockerfile \
      -t spark-base .
    
    docker build \
      -f spark-master.Dockerfile \
      -t spark-master .
    
    docker build \
      -f spark-worker.Dockerfile \
      -t spark-worker .
    
    docker build \
      --build-arg spark_version="${SPARK_VERSION}" \
      --build-arg jupyterlab_version="${JUPYTERLAB_VERSION}" \
      -f jupyterlab.Dockerfile \
      -t jupyterlab .
    ```
    
    ```docker
    # docker-compose.yml
    version: "3.6"
    volumes:
      shared-workspace:
        name: "hadoop-distributed-file-system"
        driver: local
    services:
      jupyterlab:
        image: jupyterlab
        container_name: jupyterlab
        ports:
          - 8888:8888
        volumes:
          - shared-workspace:/opt/workspace
      spark-master:
        image: spark-master
        container_name: spark-master
        ports:
          - 8080:8080
          - 7077:7077
        volumes:
          - shared-workspace:/opt/workspace
      spark-worker-1:
        image: spark-worker
        container_name: spark-worker-1
        environment:
          SPARK_WORKER_CORES: "1"
          SPARK_WORKER_MEMORY: "1024m"
        ports:
          - 8081:8081
        volumes:
          - shared-workspace:/opt/workspace
        depends_on:
          - spark-master
      spark-worker-2:
        image: spark-worker
        container_name: spark-worker-2
        environment:
          SPARK_WORKER_CORES: "1"
          SPARK_WORKER_MEMORY: "1024m"
        ports:
          - 8082:8081
        volumes:
          - shared-workspace:/opt/workspace
        depends_on:
          - spark-master
    ```
    
    ![](https://i.ibb.co/4FTwm0g/2023-06-16-10-51-31.png)
    
    ![](https://i.ibb.co/cLcZq9r/2023-06-16-10-52-15.png)
    

##### 참조
---
https://www.kdnuggets.com/2020/07/apache-spark-cluster-docker.html