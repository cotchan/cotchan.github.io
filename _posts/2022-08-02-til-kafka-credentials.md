---
layout: post
title: Kafka credentials 설정
subtitle: SASL_PLAINTEXT
categories: todayilearned
tags: [todayilearned]
---

- 개인 공부 목적으로 작성한 포스팅입니다.
- 아래 출처를 참고하여 작성하였습니다. :)

### 상황

- 브릿지 모드로 실행되는 스프링부트 컨테이너(도커)에서만 `호스트 PC의 카프카에 접근하도록` 하기 위해 `SASL` 인증을 추가하면서 공부한 내용을 정리합니다.

### JAAS/SASL

- Kafka credentials 설정을 하면서 처음 접해본 단어들입니다.

#### JAAS

- JAVA 인증 및 인가 서비스 (Java Authentication and Authorization Service)
- JAAS가 하는 일은 `사용자의 인증/인가`를 담당합니다.
  - 사용자의 인증: 현재 자바코드를 실행하고 있는 사용자가 누군인지 확실하고 안전하게 결정
  - 사용자의 인가: 수행된 행위를 하는데 필요한 ACL (접근권한제어, 권한) 을 가지고 있는지 확인

#### SASL

- `SASL`은 연결지향 프로토콜에서 교체 가능한 메커니즘을 통해 인증 및 데이터 보안 서비스를 제공하는 프레임워크

### broker 쪽 설정

- `SASL` 인증을 설정하려면 `카프카 브로커`, `카프카 클라이언트`의 코드 수정이 필요합니다.
- 먼저 카프카 브로커쪽에서 수정할 부분을 알아보겠습니다.

#### jaas.conf 파일 생성

카프카 브로커가 위치한 서버에 `kafka_jaas.conf` 파일을 생성합니다.

- `kafka_jaas.conf`라고 이름짓고 `${KAFKA_PATH}/config` 디렉토리 안에 저장해놓습니다.
- 아래 코드처럼 작성하면, 카프카 클라이언트가 사용할 수 있는 `alice`, `bob`, `charlie` 사용자가 추가됩니다.

```conf
KafkaServer {
    # 활성화 시킬 로그인 모듈이 PlainLoginModule 이라는 의미
    org.apache.kafka.common.security.plain.PlainLoginModule required
    #  이 파일을 사용하여 구동하는 애플리케이션은 본인의 정보로 위에 표시된 username 및 password를 사용
    username="admin"
    password="admin"

    # 서버가 가지고 있는 아이디/비밀번호의 리스트로, 유저가 접속할 계정 이름입니다.
    # Kafka는 user_[ID]="[비밀번호]"의 형식으로 아이디/비밀번호를 보관
    user_alice="alice"
    user_bob="bob"
    user_charlie="charlie";
};
```

#### server.properties 수정

- `"인증을 사용한다"`라는 의미로 프로토콜을 설정해줘야 합니다.

```properties
listeners=SASL_PLAINTEXT://:9092

# broker 간의 통신에 사용할 protocol을 의미 SASL 인증 방식을 처리하기 위한 설정.
security.inter.broker.protocol=SASL_PLAINTEXT

# 브로커간 통신에 사용할 SASL 메커니즘(암호화 알고리즘), 기본값은 GSSAPI
sasl.mechanism.inter.broker.protocol=PLAIN

# Kafka 서버에서 활성화 시킬 SASL 메커니즘(암호화 알고리즘)
sasl.enabled.mechanisms=PLAIN
```

- `listeners`
  - 일반적으로 `PLAINTEXT://IP:PORT` 형식을 사용합니다.
  - SASL 기반의 인증을 사용하려면 `SASL_PLAINTEXT:`로 적용해야 합니다.
  - TLS가 활성화되어 있다면 `SASL_SSL:`을 사용합니다.

#### kafka-server-start.sh 수정

- kafka 서버의 실행 파일인 `kafka-server-start.sh`를 실행할 때, 위의 `kafka_jaas.conf` 정보를 물고 실행되도록 환경 변수를 설정해줘야 합니다.
- 쉘 파일 내부에 `export KAFKA_OPTS="-Djava.security.auth.login.config=file:$base_dir/../config/kafka_jaas.conf"` 코드를 추가해줍니다.
- 그러면 kafka 브로커가 실행되면서 위의 `kafka_jaas.conf` 인증 파일을 물고 실행됩니다.

`kafka-server-start.sh에` 추가한 코드

```bash
if [ "x$KAFKA_OPTS" = "x" ]; then
  export KAFKA_OPTS="-Djava.security.auth.login.config=file:$base_dir/../config/kafka_jaas.conf"
fi
```

- `kafka_jaas.conf` 설정이 추가된 `kafka-server-start.sh`

```bash
if [ $# -lt 1 ];
then
        echo "USAGE: $0 [-daemon] server.properties [--override property=value]*"
        exit 1
fi
base_dir=$(dirname $0)

if [ "x$KAFKA_LOG4J_OPTS" = "x" ]; then
    export KAFKA_LOG4J_OPTS="-Dlog4j.configuration=file:$base_dir/../config/log4j.properties"
fi

if [ "x$KAFKA_HEAP_OPTS" = "x" ]; then
    export KAFKA_HEAP_OPTS="-Xmx1G -Xms1G"
fi

if [ "x$KAFKA_OPTS" = "x" ]; then
  export KAFKA_OPTS="-Djava.security.auth.login.config=file:$base_dir/../config/kafka_jaas.conf"
fi

EXTRA_ARGS=${EXTRA_ARGS-'-name kafkaServer -loggc'}

COMMAND=$1
case $COMMAND in
  -daemon)
    EXTRA_ARGS="-daemon "$EXTRA_ARGS
    shift
    ;;
  *)
    ;;
esac

exec $base_dir/kafka-run-class.sh $EXTRA_ARGS kafka.Kafka "$@"
```

- 여기까지가 `인증`을 위해 브로커에 필요한 추가 설정입니다.

### client 쪽 설정

- client 쪽 설정은 `producer/consumer` 모두를 의미합니다.
- 클라이언트쪽에서도 프로토콜을 설정해주고, 브로커에 접근하기 위한 `username`, `password`를 설정해줘야 합니다.

#### 예시 코드

- `인증을 사용하면,` 아래 코드처럼 클라이언트 설정에 다음 속성들이 추가됩니다.
  - `SECURITY_PROTOCOL_CONFIG`
  - `SASL_MECHANISM`
  - `SASL_JAAS_CONFIG`
- 이 포스팅에서는 자바 애플리케이션으로 `producer/consumer`를 구현했을 때 필요한 추가 설정만 다루며, 커맨드 라인툴에서 인증을 추가하는 부분은 다루지 않습니다.

```java
Properties props = new Properties();
props.put(CommonClientConfigs.BOOTSTRAP_SERVERS_CONFIG, <BROKERS>);
props.put(CommonClientConfigs.SECURITY_PROTOCOL_CONFIG, "SASL_SSL");
props.put(SaslConfigs.SASL_MECHANISM, "PLAIN");
props.put(SaslConfigs.SASL_JAAS_CONFIG, "org.apache.kafka.common.security.plain.PlainLoginModule required username=\"" + username + "\" password=\"" + password + "\";");
```

#### 기존 producer/consumer 설정

```java
private Map<String, Object> producerConfigs() {
    Map<String, Object> configs = new HashMap<>();
    configs.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
    configs.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
    configs.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, JsonSerializer.class);
    configs.put(ProducerConfig.RETRIES_CONFIG, retries);
    return configs;
}

private Map<String, Object> consumerConfigs() {
    Map<String, Object> configs = new HashMap<>();
    configs.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
    configs.put(ConsumerConfig.GROUP_ID_CONFIG, groupId);
    configs.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, offsetReset);
    return configs;
}
```

#### '인증'이 추가된 producer/consumer 설정

```java
private Map<String, Object> producerConfigs() {
    Map<String, Object> configs = new HashMap<>();
    configs.put(CommonClientConfigs.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
    configs.put(CommonClientConfigs.SECURITY_PROTOCOL_CONFIG, "SASL_PLAINTEXT");
    configs.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
    configs.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, JsonSerializer.class);
    configs.put(ProducerConfig.RETRIES_CONFIG, retries);
    configs.put(SaslConfigs.SASL_MECHANISM, "PLAIN");
    configs.put(SaslConfigs.SASL_JAAS_CONFIG, "org.apache.kafka.common.security.plain.PlainLoginModule required username=\"" + userName + "\" password=\"" + password + "\";");
    return configs;
}

private Map<String, Object> consumerConfigs() {
    Map<String, Object> configs = new HashMap<>();
    configs.put(CommonClientConfigs.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
    configs.put(CommonClientConfigs.SECURITY_PROTOCOL_CONFIG, "SASL_PLAINTEXT");
    configs.put(ConsumerConfig.GROUP_ID_CONFIG, groupId);
    configs.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, offsetReset);
    configs.put(SaslConfigs.SASL_MECHANISM, "PLAIN");
    configs.put(SaslConfigs.SASL_JAAS_CONFIG, "org.apache.kafka.common.security.plain.PlainLoginModule required username=\"" + userName + "\" password=\"" + password + "\";");
    return configs;
}
```

---

- 출처
  - [Security Considerations for SASL/SCRAM](https://docs.confluent.io/platform/current/kafka/authentication_sasl/authentication_sasl_scram.html#security-considerations-for-sasl-scram)
  - [kafka set credentials](https://qkqhxla1.tistory.com/1097)
  - [Kafka 보안 (2) - SASL/PLAIN](https://springboot.cloud/32)
  - [How to add JVM parameters to Apache Kafka?](https://stackoverflow.com/questions/40890821/how-to-add-jvm-parameters-to-apache-kafka)
  - [How to configure kafka consumer with sasl mechanism PLAIN and with security protocol SASL_SSL in java?](https://stackoverflow.com/questions/57886163/how-to-configure-kafka-consumer-with-sasl-mechanism-plain-and-with-security-prot)
  - [Java Authentication and Authorization Service (JAAS)](https://springboot.cloud/29)
