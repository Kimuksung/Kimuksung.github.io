---
layout: post
title:  "Kubernetes Airflow Scheduler 개선하기"
author: Kimuksung
categories: [ Airflow ]
#blog post image
image: assets/images/airflow.png
comments: False
featured: True
---

<br>
안녕하세요

오늘은 Kubernetes Executor 환경에서 Airflow 2.9.3을 운영하며 겪은 현상에 대해 공유 드리려고 합니다.

Airflow가 실제 동작하는 영역과 Executor 영역은 Node가 분리되어 있어 서로가 영향을 끼치지 않습니다.

Airflow Web에서 버튼이 상호작용이 안된다던지 하는 현상이 발생되었습니다.

해당 시간에 Scheduler가 꺼졌다가 켜진 이력을 발견하였고 에러는 명확하였습니다. 

<br>

### 1. 원인

원인은 아래와 같습니다.

-> The Node was low on resource: ephemeral-storage 로 인한 Evict
1. Airflow Scheduler에서 모종의 이유로 Node에서 퇴출(Pod의 생명 주기는 한달에 한번 재생성됨)
2. EmptyDir → ephemeral-storage 를 사용중
3. ephemeral-storage Pod의 임시 스토리지가 Node의 임계치보다 더 커졌기 때문에 퇴출 시켰다. ephemeral-storage 의 크기가 왜 커졌을까?
4. /opt/airflow/logs 하위에 dag_processor_manager, scheduler 로그를 적재
5. /opt/airflow/logs 크기가 2GB 넘어가면 Evict

<br>

### 2. 상세 내용

처음에 생각한 방안은 아래 정도였습니다.
1. Node와 공유되는 공간이 EmptyDir 이외의 다른 Path로 선언
2. 로그 데이터를 7일 주기적으로 삭제 처리한다. (로그가 얼마나 쌓여서 퇴출당한지 모르기 때문)
3. Node Evict 대상이 되지 않도록 한다.

잘 처리가 되었을까요?
- ephemeral-storage는 kubelet에서 퇴출을 시키기 때문에 evict 값을 false로 처리하는거랑 상관이 없습니다.
- 적용을 했음에도 불구하고 똑같이 발생하였습니다.
- 어려움이 있다면, kubernetes pod에 직접 확인할 수 있다면 좋겠지만 R&R상 접근이 불가합니다.

<br>
다음 방안을 찾아봅니다.

그래도 조금 더 명확해진 이유는 찾았습니다. 

storage가 2GB가 넘으면 퇴출 당한다는 부분과 주기적으로 삭제 되는 path가 scheduler 하위라는점을요.
이제부터는 Docker에다가 직접 테스트를 해봅니다.
4. AIRFLOW__LOGGING__DAG_PROCESSOR_MANAGER_LOG_STDOUT : 로그 적재 -> stdout로 노출
5. AIRFLOW__LOGGING__DAG_PROCESSOR_MANAGER_LOG_LOCATION : scheduler 하위로 위치를 옮겨 삭제 처리
6. logging_level를 info -> warning, error로 처리
7. logging_config class를 수정하여 삭제 처리 하도록 한다.

<br>

### 도장 깨기 시작

warning으로 처리하면 잘 해결될줄 알았으나...?

logging level을 warning으로 바꾸니 실제 task의 Logs 값들이 노출되지 않아 emr serverless id와 같은 값들을 알 수 없어 값을 식별할 수 없는 문제가 발생하였습니다. 

그래서 결국 Airflow 코드를 보고 직접 만들자로 생각했습니다.
- airflow 2.9.3은 [github](https://github.com/apache/airflow/blob/2.9.3/airflow/config_templates/airflow_local_settings.py) 내 DEFAULT_LOGGING_CONFIG 값으로 통제되는것을 확인합니다.
- 로그 class들은 하위 값들을 가져다 쓰네요 [class github](https://github.com/apache/airflow/tree/848d918aa29b147c0e6280d14092055736bac3f2/airflow-core/src/airflow/utils/log) 

DEFAULT_LOGGING_CONFIG 의 기본 값을 살펴봅니다.
- 기본적으로 console, task, processor, processor_to_stdout에 대해 처리를 하게 되어있네요.
- dag_manager_processor도 별도의 설정이 없다면 processor 기준으로 처리가 된다
- 원격 로깅 설정하는 경우
  - handler task에만 적용하는 것을 볼 수 있다. 왜일까?
  - task에 대해서는 위 동작이 적용 airflow.providers.amazon.aws.log.s3_task_handler.S3TaskHandler 
  - processor는 S3TaskHandler 적용 시 파싱이 되지 않아 실제 Webserver에서 DAG가 보이지 않음

<details>
  <summary>상세 코드</summary>

<pre><code>
DEFAULT_LOGGING_CONFIG: dict[str, Any] = {
    "version": 1,
    "disable_existing_loggers": False,
    "formatters": {
        "airflow": {
            "format": LOG_FORMAT,
            "class": LOG_FORMATTER_CLASS,
        },
        "airflow_coloured": {
            "format": COLORED_LOG_FORMAT if COLORED_LOG else LOG_FORMAT,
            "class": COLORED_FORMATTER_CLASS if COLORED_LOG else LOG_FORMATTER_CLASS,
        },
        "source_processor": {
            "format": DAG_PROCESSOR_LOG_FORMAT,
            "class": LOG_FORMATTER_CLASS,
        },
    },
    "filters": {
        "mask_secrets": {
            "()": "airflow.utils.log.secrets_masker.SecretsMasker",
        },
    },
    "handlers": {
        "console": {
            "class": "airflow.utils.log.logging_mixin.RedirectStdHandler",
            "formatter": "airflow_coloured",
            "stream": "sys.stdout",
            "filters": ["mask_secrets"],
        },
        "task": {
            "class": "airflow.utils.log.file_task_handler.FileTaskHandler",
            "formatter": "airflow",
            "base_log_folder": os.path.expanduser(BASE_LOG_FOLDER),
            "filters": ["mask_secrets"],
        },
        "processor": {
            "class": "airflow.utils.log.file_processor_handler.FileProcessorHandler",
            "formatter": "airflow",
            "base_log_folder": os.path.expanduser(PROCESSOR_LOG_FOLDER),
            "filename_template": PROCESSOR_FILENAME_TEMPLATE,
            "filters": ["mask_secrets"],
        },
        "processor_to_stdout": {
            "class": "airflow.utils.log.logging_mixin.RedirectStdHandler",
            "formatter": "source_processor",
            "stream": "sys.stdout",
            "filters": ["mask_secrets"],
        },
    },
    "loggers": {
        "airflow.processor": {
            "handlers": ["processor_to_stdout" if DAG_PROCESSOR_LOG_TARGET == "stdout" else "processor"],
            "level": LOG_LEVEL,
            # Set to true here (and reset via set_context) so that if no file is configured we still get logs!
            "propagate": True,
        },
        "airflow.task": {
            "handlers": ["task"],
            "level": LOG_LEVEL,
            # Set to true here (and reset via set_context) so that if no file is configured we still get logs!
            "propagate": True,
            "filters": ["mask_secrets"],
        },
        "flask_appbuilder": {
            "handlers": ["console"],
            "level": FAB_LOG_LEVEL,
            "propagate": True,
        },
    },
    "root": {
        "handlers": ["console"],
        "level": LOG_LEVEL,
        "filters": ["mask_secrets"],
    },
}
</code></pre>


</details>

<br>

즉, Airflow Scheduler는 3가지의 로그로 동작하는거 같네요.
1. dag_processor_manager
2. task
3. processor

Airflow Config에서 말하는 설정 값들은 task를 제외하고는 3개를 제어할 수 없도록 되어있다.
- 그렇기에 적용을 위해서는 local_settings 값을 가져와서 별도로 설정을 해준다.
- 목표는 task를 제외한 로깅을 최소화하거나 주기적으로 지울 수 있도록 하는것으로 설정값을 제어해서 loggin level을 상세하게 제어해준다. info -> warning
- 이 때, dag_processor_manager 의 파일 크기를 제어하기 위해 RotatingFileHandler 를 사용하는데 이건 이미 파일이 있어야하기 때문에 미리 생성을 해주어야한다.

<details>
  <summary>상세 코드</summary>

<pre><code>
AIRFLOW__LOGGING__LOGGING_CONFIG_CLASS: 'config.airflow_local_settings.LOGGING_CONFIG'
AIRFLOW__LOGGING__REMOTE_LOGGING: 'true'
AIRFLOW__LOGGING__DELETE_LOCAL_LOGS: 'true'
AIRFLOW__LOGGING__REMOTE_BASE_LOG_FOLDER: "s3://bucket/prefix"
</code></pre>

<pre><code>
# config.airflow_local_settings.py
from copy import deepcopy
from airflow.config_templates.airflow_local_settings import DEFAULT_LOGGING_CONFIG
import os
from airflow.configuration import conf
from logging.handlers import RotatingFileHandler

LOGGING_CONFIG = deepcopy(DEFAULT_LOGGING_CONFIG)

BASE_LOG_FOLDER = conf.get('logging', 'BASE_LOG_FOLDER')
DAG_PROCESSOR_MANAGER_LOG = os.path.join(BASE_LOG_FOLDER, 'dag_processor_manager', 'dag_processor_manager.log')
DEFAULT_MAX_BYTES = 1 * 1024 * 1024  # 1MB
DEFAULT_BACKUP_COUNT = 1
REMOTE_BASE_LOG_FOLDER: str = conf.get_mandatory_value("logging", "REMOTE_BASE_LOG_FOLDER")

# processor_manager 설정
LOGGING_CONFIG['handlers']['dag_processor_manager_handler'] = {
    "class": "logging.handlers.RotatingFileHandler",
    "formatter": "airflow",
    "filename": DAG_PROCESSOR_MANAGER_LOG,
    "maxBytes": DEFAULT_MAX_BYTES,
    "backupCount": DEFAULT_BACKUP_COUNT,
}

LOGGING_CONFIG['loggers']['airflow.processor_manager'] = {
    "handlers": ["dag_processor_manager_handler"],
    "level": "WARNING",
    "propagate": True
}
# processor 설정
LOGGING_CONFIG['loggers']['airflow.processor']['level'] = "WARNING"

# for_test
REMOTE_LOGGING: bool = conf.getboolean("logging", "remote_logging")
delete_local_copy = conf.getboolean("logging", "delete_local_logs")
</code></pre>

</details>
<br>


