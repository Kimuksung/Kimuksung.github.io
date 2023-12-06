---
layout: post
title:  "Prometheus+Grafana로 인스턴스 모니터링"
author: Kimuksung
categories: [ Prometheus, Grafana ]
tags: [ Prometheus, Grafana ]
image: assets/images/monitoring.png
comments: False
---

안녕하세요.
그동안 감기와 차세대 업무로 인하여, 블로그 작성을 간만에 해봅니다.

금일 진행한 내용은 Hadoop/Kafka 등의 인스턴스를 띄워두었지만, 이 서비스들이 제대로 동작하고 있는지 얼마나 많은 리소스가 드는지를 대시보드로 보기 위해 진행하였습니다.

Docker-compose를 사용하였으며, 간단한 리소스들을 보여주는 용도만 사용하고 있습니다.  
Mongodb Atlas와 Aws RDS를 Cloudwatch로 모니터링해보려고 하였으나, Exporter를 설치하지 않으면 대시보드를 구성할 수 없다는 점과 Cloudwatch는 pull할 때마다, 많은 비용이 발생된다고 하여 진행하지 못하였습니다.  

그러면 Node 리소스를 대시보드로 구성해보겠습니다.

<br>

##### node_exporter
---
- url 접근 정보
- http://localhost:9100/
- http://localhost:9100/metrics

- mac에서 진행하기

```bash
$ brew install node_exporter
$ brew services start node_exporter
```

- Ubuntu Instance에 실행하기

```bash
$ wget https://github.com/prometheus/node_exporter/releases/download/v1.5.0/node_exporter-1.5.0.linux-amd64.tar.gz
$ tar xvfz node_exporter-*.*-amd64.tar.gz
$ cd node_exporter-*.*-amd64
$ ./node_exporter
```

- 자동화

```bash
$ sudo vi /etc/systemd/system/node_exporter.service

[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=root
Group=root
Type=simple
# 실행 경로 추가
ExecStart=/home/ubuntu/node_exporter-1.5.0.linux-amd64/node_exporter

[Install]
WantedBy=multi-user.target

$ sudo systemctl daemon-reload
$ sudo systemctl enable node_exporter.service
$ sudo systemctl start node_exporter.service
```

<br>

##### Prometheus
---
- http://localhost:9090/

```bash
$ brew install prometheus
$ brew services start prometheus
```

- prometheus.yml 을 통한 정보 입력

```bash
$ sudo vi /opt/homebrew/etc/prometheus.yml
```

```bash
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "prometheus"
    static_configs:
    - targets: ["localhost:9100"]
```

```bash
# restart
$ brew services restart prometheus
```

- docker 간단 실행

```bash
$ docker run -p 9090:9090 -v /Users/andrew/dev/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus
```

- docker-compose 구성
    - docker-compose.yml
    - prometheus.yml

```yaml
# docker-compose.yml
version: '3'

services:
  prometheus:
    image: prom/prometheus
    container_name: prometheus
    volumes:
      - /Users/wclub/monitoring/prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - 9090:9090
    networks:
      - prometheus-net

networks:
  prometheus-net:
    driver: bridge
```

```yaml
# prometheus.yml
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

scrape_configs:
  - job_name: 'node-exporter'
    static_configs:
      - targets: ['ip:port']
  - job_name: 'kafka'
    static_configs:
      - targets: ['ip:port']
  - job_name: 'local'
    static_configs:
      - targets: ['local:9100', 'host.docker.internal:9100']
  - job_name: 'hadoop'
    static_configs:
      - targets: ['ip:port']
        labels:
          hadoop: 'master'
  - job_name: 'redash'
    static_configs:
      - targets: ['ip:port']
```

<a href="https://postimg.cc/0rQgC2Ss" target="_blank"><img src="https://i.postimg.cc/PfbtTCrP/Untitled-77.png" alt="Untitled-77"/></a>

<a href="https://postimg.cc/CzBWWPJr" target="_blank"><img src="https://i.postimg.cc/zX08H4hN/Untitled-78.png" alt="Untitled-78"/></a>

<br>

##### Grafana

---

- docker-compose.yml

```yaml
grafana:
    image: grafana/grafana
    container_name: grafana
    # user: admin:admin
    ports:
      - 3000:3000 # 접근 포트 설정 (컨테이너 외부:컨테이너 내부)
    volumes:
      - /Users/wclub/monitoring/grafana:/var/lib/grafana
    restart: always
    networks:
      - prometheus-net
```

대시보드 구성하기

---

- node exporter 구성
- https://grafana.com/grafana/dashboards/1860-node-exporter-full/ 툴 가져와 사용

<a href="https://postimg.cc/7fFpD3WH" target="_blank"><img src="https://i.postimg.cc/GmtrFzPy/Untitled-79.png" alt="Untitled-79"/></a>

<br>

##### 참조
---
- https://hippogrammer.tistory.com/258
- https://www.devkuma.com/docs/prometheus/docker-compose-install/