---
layout: post
title: Kafka와 RabbitMQ 비교
subtitle: Message Queue와 Pub/Sub 방식
categories: todayilearned
tags: [todayilearned]
---

- 개인 공부 목적으로 작성한 포스팅입니다.
- 아래 출처를 참고하여 작성하였습니다. :)

### Kafka

#### Kafka 특징

1. Pull 방식

> 파티션들이 소비자(Consumer)로 메세지를 전달하는 것이 아니라, 소비자(Consumer)가 `pulling`을 통해 `데이터를 가져가서 처리`합니다.

2. 메시지 보존

> Kafka는 Consumer가 메세지를 소비했는지 여부와 상관없이, 미리 `정해진 만료시간까지 파티션에 메세지를 보관`합니다
> 따라서 `메시지 재사용`이 가능하며, Consumer가 과거 메세지를 쉽게 다시 읽을 수 있습니다.
> 그러므로 이벤트 소싱이나, 감사 로그와 같은 메커니즘을 구현하기에 적절합니다.

3. 메시지 순서 보장

> 같은 토픽 파티션으로 보내진 메세지는 `순서대로 처리됨을 보장`합니다.

4. 대용량 스트림 처리

### RabbitMQ

#### RabbitMQ 특징

1. Push 방식

> 메시지 브로커가 컨슈머에게 메시지를 Push 해주는 방식입니다.

2. `Message acknowledgements(Delivery Guarantees)`

> RabbitMQ는 Producer와 Consumer 사이의 Message 전달을 보장하기 위한 기법으로 ACK를 제공합니다.
> ACK 기법은 Message가 최소 한번 이상은 전달되는 것을 보장합니다.
> 송신자는 Message를 전송한 이후에 수신자로부터 ACK를 받지 못한다면, 수신자에게 ACK를 받을때까지 반복해서 Message를 전송해야 합니다.
> 따라서 Producer는 ACK를 받지 못하면 Message를 재전송 하도록 구현되어 있습니다.
> 그래서 수신자는 동일한 Message를 2번이상 받을 수 있습니다.

3. Message priorities

> RabbitMQ에서는 `메시지 우선 순위`를 지정하고 `우선 순위가 높은 메시지를 먼저 소비`할 수 있습니다.

### Kafka vs RabbitMQ

둘의 차이가 발생하는 지점을 정리합니다.

#### Message ordering

Kafka

- 같은 토픽 파티션으로 보내진 메세지는 `순서대로 처리됨을 보장`합니다.
- 만약 여러개의 파티션 사이에서 순서를 보장하고 싶다면, 키를 이용해서 메세지를 그룹화하여 `동일한 키를 가진 메세지`는 `동일한 파티션으로 이동하도록` 해야한다.

RabbitMQ

- Not supported.
- 메세지 소비자가 하나라면, 메시지 순서를 보장한다. 그러나 메시지를 읽는 여러 소비자가 있으면 메시지 처리 순서에 대해 보장할 수 없다.

#### Message lifetime

Kafka

- Kafka는 기본적으로 메시지를 보존하는 `로그`입니다. `보존 정책`을 지정하여 이를 관리할 수 있습니다.
- Kafka는 Consumer가 메세지를 소비했는지 여부와 상관없이, 미리 `정해진 만료시간까지 파티션에 메세지를 보관`합니다

RabbitMQ

- RabbitMQ는 대기열이므로 메시지는 `한 번 소비되면 사라지고` ACK가 제공됩니다.

#### Delivery Guarantees

Kafka

- 한 번 토픽에 저장된 메시지는 유실되지 않습니다.

RabbitMQ

- RabbitMQ는 강력하고 안정적이고 내구성 있는 메시징 보장을 제공합니다.
- 배송 보증은 다음을 통해 달성됩니다.
  - `Message durability` - RabbitMQ에 한 번 저장된 메시지를 잃지 않음
  - `Message acknowledgements` - RabbitMQ와 게시자/구독자(Producer/Consumer) 간의 Signal

#### Message priorities

Kafka

- Not supported.

RabbitMQ

- RabbitMQ에서는 `메시지 우선 순위`를 지정하고 `우선 순위가 높은 메시지를 먼저 소비`할 수 있습니다.

#### Pull vs Push Approach

Kafka

- 파티션들이 소비자(Consumer)로 메세지를 전달하는 것이 아니라, 소비자(Consumer)가 `pulling`을 통해 `데이터를 가져가서 처리`합니다.

RabbitMQ

- 메시지 브로커가 컨슈머에게 메시지를 Push 해주는 방식입니다.

### Kafka, RabbitMQ 장/단점

---

- 출처
  - [Kafka vs. RabbitMQ: Architecture, Performance & Use Cases](https://www.upsolver.com/blog/kafka-versus-rabbitmq-architecture-performance-use-case)
  - [RabbitMQ와 Kafka 비교하기](https://escapefromcoding.tistory.com/705#kafka)
  - [RabbitMQ ACK](https://ssup2.github.io/theory_analysis/RabbitMQ_Ack/)
  - [RabbitMQ vs Kafka Part 4 - Message Delivery Semantics and Guarantees](https://jack-vanlightly.com/blog/2017/12/15/rabbitmq-vs-kafka-part-4-message-delivery-semantics-and-guarantees)
