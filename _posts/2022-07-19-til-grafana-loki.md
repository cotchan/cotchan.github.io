---
layout: post
title: Grafana Loki 적용기
subtitle: SpringBoot 로그 모니터링 방법
categories: todayilearned
tags: [todayilearned]
---

- 개인 공부 목적으로 작성한 포스팅입니다.
- 아래 출처를 참고하여 작성하였습니다. :)

### 상황

- 도커 컨테이너 안에 file로 로그를 쌓아두다가 서버가 죽어버리니 컨테이너도 죽어버렸습니다.
- 컨테이너가 죽어버리니 내부 로그를 볼 수 있는 방법이 없었습니다ㅠㅠ
- 로그를 컨테이너 외부에 쌓아두지 않은 점을 반성하면서 이어서 든 생각은, 서버에 쌓아둬봤자 이전처럼 서버에 문제가 생기면 `문제 상황일 때` 또 로그를 못 보는 건 매한가지가 아닌가 싶었습니다.
- 그래서 `서버 외부에 로그를 쌓는 법`을 리서치하면서 `Grafana Loki`를 적용한 내용을 정리합니다. :)
- 단, 위 방식을 적용하기 위해 저는 `도커 볼륨`을 사용했고, 해당 부분은 생략하고 `Grafana Loki`를 적용한 부분만 포스팅을 남깁니다.

### Grafana Loki

#### Promtail

- `애플리케이션의 로그를 수집`하여 Loki에 보내는 `로그 수집기`입니다.

#### Loki

- `로그 저장 및 구문 분석에 사용`되며 다운스트림 프레젠테이션을 위한 쿼리 API를 제공합니다.

#### Grafana

- Grafana는 Loki의 `로그 시각화를 담당`합니다.

### 설치

- 다운로드 받을 때는 master 브랜치가 아니라 `버전명이 제대로 기입`되어야 합니다.
- 저는 참고한 포스팅을 따라 `v2.3.0`으로 진행했습니다.

#### Promtail,Loki 로컬 설치

```bash
wget https://github.com/grafana/loki/releases/download/v2.3.0/loki-linux-amd64.zip
wget https://github.com/grafana/loki/releases/download/v2.3.0/promtail-linux-amd64.zip
```

#### Promtail,Loki 설정 파일 설치

```bash
wget https://raw.githubusercontent.com/grafana/loki/v2.3.0/clients/cmd/promtail/promtail-local-config.yaml
wget https://raw.githubusercontent.com/grafana/loki/v2.3.0/cmd/loki/loki-local-config.yaml
```

### 설정

- 설치를 했으니 필요한 설정을 해줍니다.

#### Promtail 설정

- 초기의 `promtail-local-config.yaml` 아래와 같습니다.
- `clients`에는 `실행중인 Loki url`을 입력합니다.
- `loki/api/v1/push`는 `promtail`로 수집한 로그를 Loki에 푸쉬하는 POST 요청 api 입니다.

```yml
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
  - job_name: system
    static_configs:
      - targets:
          - localhost
        labels:
          job: varlogs
          __path__: /var/log/*log
```

- `scrape-configs`에 job을 추가할 수 있어서 저는 아래와 같이 수정했습니다.
- 배포서버 내에 `loki`와 `Promtail`이 같이 있었으므로 `clients` 속성은 localhost로 주었습니다.
- `/tmp/log/*log`: 제가 SpringBoot에서 로그를 내보내는 경로가 `/tmp/log`이며 `*log`를 사용해서 해당 경로 안에 있는 모든 `*.log` 파일을 읽을 수 있습니다.

```yaml
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://localhost:3100/loki/api/v1/push

scrape_configs:
  - job_name: server-log
    static_configs:
      - targets:
          - localhost
        labels:
          job: server-log
          __path__: /tmp/log/*log
```

- 다른 서버에서 실행중인 Loki로 보내는 방법은 아래와 같습니다.

```yaml
clients:  # url 을 변경
  - url: http://{요청 보낼 Loki 서버 url}:{요청 보낼 Loki port}/loki/api/v1/push

scrape_configs:
  - job_name: mylog
  static_configs:
  - targets:
    - {요청 보낼 Loki 서버 url}
  labels:
    job: mylog
    __path__: {경로}/mylog.log
```

#### Loki 설정

- `loki-local-config.yaml` 설정을 건드리면 되는데 따로 건드린 건 없었습니다.
- 포트 변경을 원한다면 `http_listen_port`를 원하는 포트로 바꿔주면 됩니다. 디폴트 포트는 `3100`입니다.

```yaml
auth_enabled: false

server:
  http_listen_port: 3100
  grpc_listen_port: 9096
  
# ...
```

### 실행

- 이제 모든 설정을 마쳤으니 `Loki`와 `Promtail`을 실행하면 됩니다.

#### Loki 실행

- 아래 명령어를 사용하면 `loki`가 실행됩니다.

```bash
./loki-linux-amd64 -config.file=loki-local-config.yaml
```

```bash
# 백그라운드로 띄우기
nohup ./loki-linux-amd64 -config.file=loki-local-config.yaml &
```

#### Promtail 실행

- 아래 명령어를 사용하면 `promtail`가 실행되며 로깅 수집 대상이 되는 파일을 연결해줍니다.

```bash
./promtail-linux-amd64 -config.file=promtail-local-config.yaml
```

```bash
# 백그라운드로 띄우기
nohup ./promtail-linux-amd64 -config.file=promtail-local-config.yaml &
```

### grafana 설정

---

- 출처
  - [Spring Boot 모니터링 적용 - 2](https://jujeol-jujeol.github.io/2021/10/28/Spring-Boot-%EB%AA%A8%EB%8B%88%ED%84%B0%EB%A7%81-%EC%A0%81%EC%9A%A9-2)
  - [SpringBoot integration of lightweight logging system loki - 1](https://www.springcloud.io/post/2022-02/springboot-loki-1/#gsc.tab=0)
  - [SpringBoot integration of lightweight logging system loki - 2](https://www.springcloud.io/post/2022-02/springboot-loki-2/#gsc.tab=0)
  - [Logging/Grafana Loki 통한 로그 수집하기](https://medium.com/@dudwls96/logging-grafana-loki-%ED%86%B5%ED%95%9C-%EB%A1%9C%EA%B7%B8-%EC%88%98%EC%A7%91%ED%95%98%EA%B8%B0-d57ba1b75ab3)
  - [Install and run Grafana Loki locally](https://grafana.com/docs/loki/latest/installation/local/)
