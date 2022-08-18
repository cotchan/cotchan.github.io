---
layout: post
title: Kafka 커맨드 라인툴 SASL_PLAINTEXT 연결 설정
subtitle: 카프카 커맨드 라인툴에서 SASL_PLAINTEXT 프로토콜로 연결하는 방법
categories: todayilearned
tags: [todayilearned]
---

- 개인 공부 목적으로 작성한 포스팅입니다.
- 아래 출처를 참고하여 작성하였습니다. :)

### 상황

- 브로커에 `SASL_PLAINTEXT` 프로토콜을 설정하면서, 로컬 호스트에서 `커맨드 라인툴`을 사용할 때도 인증 정보를 제공해줘야 했습니다.
- 사용하고자 했던 커맨드 라인툴 명령어는 `토픽 생성 명령어`와 `브로커가 제대로 동작하는 지 확인하는 명령어`였습니다.
- 그래서 커맨드 라인툴에서 `SASL_PLAINTEXT` 프로토콜을 사용하기 위해 적용했던 내용을 정리합니다.

### consumer.properties 설정

- `${KAFKA_HOME}/config/consumer.properties` 파일에 아래 설정을 추가해줍니다.

```properties
security.protocol=SASL_PLAINTEXT
sasl.mechanism=PLAIN
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="USER_NAME" password="USER_PASSWORD";
```

### --command-config 실행 옵션

- `SASL_PLAINTEXT` 프로토콜을 사용한다면 커맨드 실행 시 인증 정보를 함께 제공해줘야 합니다.
- 위와 같이 `consumer.properties` 파일을 수정하고 `--command-config` 옵션을 사용하면, `SASL_PLAINTEXT` 프로토콜을 사용해서 카프카 커맨드 라인툴 명령어를 사용할 수 있습니다.

다음으로는 카프카 커맨드 라인툴 명령어와 `--command-config` 옵션을 함께 쓰는 예시를 보겠습니다.

#### 토픽 생성 명령어

- 카프카 커맨드 라인툴을 사용해서 토픽을 생성하는 명령어

```bash
./kafka-topics.sh --create --bootstrap-server BROKER:9092 --topic YOUR.TOPIC.NAME --command-config ../config/consumer.properties
```

#### 브로커 상태 확인 명령어

- 특정 BROKER가 제대로 동작하고 있는지 확인하는 명령어

```bash
./kafka-broker-api-versions.sh --bootstrap-server BROKER:9092 --command-config ../config/consumer.properties
```

---

- 출처
  - [How to access SASL configure kafka from Kafka cli](https://stackoverflow.com/questions/63488557/how-to-access-sasl-configure-kafka-from-kafka-cli)
  - [Authentication using SASL/PLAIN](https://kafka.apache.org/documentation/#security_sasl_plain)
  - [security_sasl_plain_clientconfig](https://kafka.apache.org/documentation/#security_sasl_plain_clientconfig)
