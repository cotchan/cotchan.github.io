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

**Pull 방식**

- 파티션이 Consumer에게 메세지를 전달하는 게 아니라, Consumer가 `pulling`을 통해 `데이터를 가져감`
- 컨슈머가 브로커로부터 메시지를 직접 가져가는 PULL 방식
  - 그래서 컨슈머는 자신의 처리 능력만큼만 메시지를 가져와 최적 성능

**메시지 보존**

- Consumer의 메세지 소비여부와 상관없이, 미리 `정해진 만료시간까지 파티션에 메세지를 보관`합니다.
- 따라서 `메시지 재사용`이 가능하며, Consumer가 과거 메세지를 쉽게 다시 읽을 수 있습니다.
- 그러므로 이벤트 소싱이나, 감사 로그와 같은 메커니즘을 구현하기에 적절합니다.

**메시지 순서 보장**

- 같은 토픽 파티션으로 보내진 메세지는 `순서대로 처리됨을 보장`합니다.

**대용량의 분산 로그 트래픽 처리에 유리**

> 대용량 스트림 처리에 유리

- 동시에 들어오는 많은 데이터를 `여러개의 파티션에 나누어 저장`하기 때문에 `병렬로 빠르게 처리`할 수 있다.
  - Producer, Consumer 모두에 해당하는 내용

**메시지를 파일에 저장**

- 카프카를 재시작해도 메시지 유실 우려가 감소

### RabbitMQ

#### RabbitMQ 특징

**Push 방식**

- 메시지 브로커가 `컨슈머에게 메시지를 Push` 해주는 방식입니다.

**Message acknowledgements**

- RabbitMQ는 Producer와 Consumer 사이의 Message 전달을 보장하기 위한 기법으로 `ACK를 제공`합니다.
- ACK 기법은 Message가 최소 한번 이상은 전달되는 것을 보장합니다.
- 송신자는 Message를 전송한 이후에 수신자로부터 ACK를 받지 못한다면, 수신자에게 ACK를 받을때까지 반복해서 Message를 전송해야 합니다.
  - 따라서 Producer는 ACK를 받지 못하면 Message를 재전송 하도록 구현되어 있습니다.
  - 그래서 수신자는 동일한 Message를 2번이상 받을 수 있습니다.

**Message priorities**

- RabbitMQ에서는 `메시지 우선 순위`를 지정하고 `우선 순위가 높은 메시지를 먼저 소비`할 수 있습니다.

**메시지를 메모리에 저장**

### Kafka와 RabbitMQ 차이점

> Kafka와 RabbitMQ가 차이가 생기는 지점을 정리합니다.

#### Message ordering

**Kafka**

- 같은 토픽 파티션으로 보내진 메세지는 `순서대로 처리됨을 보장`합니다.
- 만약 여러개의 파티션 사이에서 순서를 보장하고 싶다면, 키를 이용해서 메세지를 그룹화하여 `동일한 키를 가진 메세지`는 `동일한 파티션으로 이동하도록` 해야합니다.

**RabbitMQ**

- Not supported.

#### Message lifetime

**Kafka**

- Kafka는 기본적으로 메시지를 보존하는 `로그`입니다. `보존 정책`을 지정하여 이를 관리할 수 있습니다.
- Consumer의 메세지 소비여부와 상관없이, 미리 `정해진 만료시간까지 파티션에 메세지를 보관`합니다.

**RabbitMQ**

- RabbitMQ는 대기열이므로 메시지는 `한 번 소비되면 사라지고` ACK가 제공됩니다.

#### Delivery Guarantees

**Kafka**

- 한 번 토픽에 저장된 메시지는 유실되지 않습니다.

**RabbitMQ**

- RabbitMQ는 강력하고 안정적이고 내구성 있는 메시징 보장을 제공합니다.
- 배송 보증은 다음을 통해 달성됩니다.
  - `Message durability` - RabbitMQ에 한 번 저장된 메시지를 잃지 않음
  - `Message acknowledgements` - RabbitMQ와 게시자/구독자(Producer/Consumer) 간의 Signal

#### Message priorities

**Kafka**

- Not supported.

**RabbitMQ**

- RabbitMQ에서는 `메시지 우선 순위`를 지정하고 `우선 순위가 높은 메시지를 먼저 소비`할 수 있습니다.

#### Pull vs Push Approach

**Kafka**

- 파티션이 Consumer에게 메세지를 전달하는 게 아니라, Consumer가 `pulling`을 통해 `데이터를 가져감`
- 컨슈머가 브로커로부터 메시지를 직접 가져가는 PULL 방식
  - 그래서 컨슈머는 자신의 처리 능력만큼만 메시지를 가져와 최적 성능

**RabbitMQ**

- 메시지 브로커가 `컨슈머에게 메시지를 Push` 해주는 방식입니다.

### Kafka, RabbitMQ 장/단점

#### Kafka 장점

- 메시지 보존
- 엄격한 메세지 순서관리
  - 과거 메시지 재생 가능성을 포함하여 장기간 메시지 보존
- 대용량의 분산 로그 트래픽 처리에 유리
  - 동시에 들어오는 많은 데이터를 `여러개의 파티션에 나누어 저장`하기 때문에 `병렬로 빠르게 처리`할 수 있다.

#### Kafka 단점

- Consumer에게 메시지 전달 확인 X
  - `실패 후 메시지를 다시 시도해야 하는 책임`은 `Consumer`에게 있으므로 Producer는 메시지가 Consumer에게 성공적으로 전달되었는지 여부를 신경 쓰지 않음

#### RabbitMQ 장점

- RabbitMQ는 Consumer의 확인 메시지가 보장됨(ACK를 제공)
  - 고로 RabbitMQ는 높은 처리량보다는 `지정된 수신인에게` 원하는 방식으로 `메시징을 신뢰성 있게 전달`
- RabbitMQ에서는 `메시지 우선 순위`를 지정하고 `우선 순위가 높은 메시지를 먼저 소비`할 수 있습니다.

#### RabbitMQ 단점

- 메시지 보존 X

---

- 출처
  - [Kafka vs. RabbitMQ: Architecture, Performance & Use Cases](https://www.upsolver.com/blog/kafka-versus-rabbitmq-architecture-performance-use-case)
  - [RabbitMQ와 Kafka 비교하기](https://escapefromcoding.tistory.com/705#kafka)
  - [RabbitMQ ACK](https://ssup2.github.io/theory_analysis/RabbitMQ_Ack/)
  - [RabbitMQ vs Kafka Part 4 - Message Delivery Semantics and Guarantees](https://jack-vanlightly.com/blog/2017/12/15/rabbitmq-vs-kafka-part-4-message-delivery-semantics-and-guarantees)
