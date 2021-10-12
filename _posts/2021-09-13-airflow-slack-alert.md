---
layout: single
classes: wide
title: "Airlflow DAG 실패/성공 시 Slack 채널로 알림 메시지 전송 연동"
last_modified_at: 2021-09-13
description: "Airflow DAG 실패(또는 성공) 시 특정 Slack 채널로 메시지 전송하기 위해서 간단하게 Slack 설정 및 Airflow Python API를 이용해 연동 해보았습니다."
categories:
  - dev
tags:
  - python, airlfow, slack, alert, alarm, dag
---

Airflow DAG 실패(또는 성공) 시 특정 Slack 채널로 메시지 전송하기 위해서 간단하게 Slack 설정 및 Airflow Python API를 이용해 연동 해보았습니다.


## Slack 설정

### App 생성

- [https://api.slack.com/](https://api.slack.com/) 접속

- App 이름과 workspace 지정

- 여기서는 Airflow test 라는 이름의 APP을 생성함

### Bot 설정

- Bots 클릭하여 Bot User 생성

- Bot Token Scopes에 아래 권한 추가

  - chat:write

- Bot token 생성

  - Install App to Workspace 클릭

### 채널에 App 추가

- Slack 채널 조인 후 채널 설정에서 아래와 같이 앱 추가
![slack app to channel](/assets/images/2021-09-13_airflow_slack_1.png?raw=true)


## MWAA local runner

- AWS MWAA를 로컬 환경에서 구동하기 위해 AWS에서 제공하는 Git 리파지토리로 Docker를 이용합니다.

### Git clone

```bash
git clone https://github.com/aws/aws-mwaa-local-runner.git
```

### 샘플 코드 추가

- slack_alert.py

```python
from airflow.operators.slack_operator import SlackAPIPostOperator


class SlackAlert:
    def __init__(self, channel, token):
        self.slack_channel = channel
        self.slack_token = token

    def _make_message(self, context):
        return (
            f'* `DAG`:  {context.get("task_instance").dag_id}'
            f'\n* `Task`:  {context.get("task_instance").task_id}'
            f'\n* `Run ID`:  {context.get("run_id")}'
            f'\n* `Execution Time`:  {context.get("execution_date")}'
        )

    def _make_attachments(self, context, title="Failure", color="danger"):
        return [
            {
                "mrkdwn_in": ["text"],
                "title": title,
                "text": self._make_message(context),
                "actions": [
                    {
                        "type": "button",
                        "name": "view log",
                        "text": "View log",
                        "url": context.get("task_instance").log_url,
                        "style": "danger" if color == "danger" else "default",
                    },
                ],
                "color": color,  # 'good', 'warning', 'danger', or hex ('#439FE0')
                "fallback": "details",  # Required plain-text summary of the attachment
            }
        ]

    def send_alert(self, context, task_id, text, title, color):
        if not self.slack_token:
            print(f"Error: send_alert - slack_token is null.")
            return

        try:
            SlackAPIPostOperator(
                task_id=task_id,
                channel=self.slack_channel,
                token=self.slack_token,
                text=text,
                attachments=self._make_attachments(context, title, color),
            ).execute(context=context)
        except Exception as e:
            print(f"Error: SlackAPIPostOperator, {str(e)}")


# token을 직접 입력하거나 아니면 Airlfow Variables에 등록된 값을 사용함
def send_fail_alert(context, channel="#airflow_alert_test", token=None):
    token = token if token else Variable.get("slack_app_token", default_var=None)
    alert = SlackAlert(channel=channel, token=token)
    alert.send_alert(
        context,
        task_id="send_fail_alert",
        text="Failure",
        title="Failure",
        color="danger",
    )


def send_success_alert(context, channel="#airflow_alert_test", token=None):
    token = token if token else Variable.get("slack_app_token", default_var=None)
    alert = SlackAlert(channel=channel, token=token)
    alert.send_alert(
        context,
        task_id="send_success_alert",
        text="Success",
        title="Success",
        color="good",
    )
```    

- test_alert.py (DAG)

```python
from airflow import DAG
from airflow.operators.bash_operator import BashOperator
from datetime import datetime, timedelta

from slack_alert import SlackAlert


alert = SlackAlert()

default_args = {
    "owner": "airflow",
    "depends_on_past": False,
    "start_date": datetime(2015, 6, 1),
    "email": ["airflow@airflow.com"],
    "email_on_failure": False,
    "email_on_retry": False,
    "retries": 0,
    "retry_delay": timedelta(minutes=5),
    "on_failure_callback": alert.send_fail_alert,
    "on_success_callback": alert.send_success_alert,
}

dag = DAG("test_alert", default_args=default_args, schedule_interval=timedelta(1))

t1 = BashOperator(task_id="fail", bash_command="exit 1", dag=dag)
# t1 = BashOperator(task_id="success", bash_command="exit 0", dag=dag)

t2 = BashOperator(task_id="print_date", bash_command="date", dag=dag)

t1 >> t2
```


## Airflow 실행

### 이미지 빌드 및 컨테이너 실행

```bash
# Docker 이미지 빌드
./mwaa-local-env build-image

 # container 실행
./mwaa-local-env start
```

### 웹 UI 접속 및 DAG 실행

- http://0.0.0.0:8080/ 접속 후 로그인

  - ID/PW: admin/test

- test_alert DAG 수동 실행 (trigger)

### Troubleshooting

- ID/PW 정상적으로 입력했는데도 로그인 화면에서 이동 안 될 경우

  - https://github.com/aws/aws-mwaa-local-runner/issues/10

- No module not found ... 에러 시

  - dags/requirements.txt에 아래 패키지 추가

  - `apache-airflow-providers-slack==2.0.0`

- Airflow Variable 추가 후 docker 재시작 시 invalid 에러 시

  - docker/script/docker-compose-local.yml에 services: local-runner: environment:내 아래 항목 추가

  - `AIRFLOW__CORE__FERNET_KEY='81HqDtbqAywKSOumSha3BhWNOdQ26slT6K0YaZeZyPs='`


## 연동 테스트

### DAG task 실패 시 Slack 채널 내 alert 메시지 전송 결과 화면
- DAG task 성공 시에도 메시지 전송하도록 추가 가능하나 그럴 경우 task 마다 별도로 메시지를 전송하게 됨

![slack app to channel](/assets/images/2021-09-13_airflow_slack_2.png?raw=true)

## 참고 자료

### MWAA local runner
- [[AWS] MWAA local runner](https://github.com/aws/aws-mwaa-local-runner)

### Airflow - Slack 연동 관련
- [[Airflow] Slack으로 결과 전달하기](https://moons08.github.io/programming/airflow-slack/)

### Slack 설정 관련
- [[Slack] OAuth Token 생성하기](https://miaow-miaow.tistory.com/148)
