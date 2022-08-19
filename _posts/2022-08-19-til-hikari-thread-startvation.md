---
layout: post
title: HikariPool - Thread starvation or clock leap detected 오류
subtitle: 인스턴스 상태검사 실패, Thread Starvation 현상
categories: todayilearned
tags: [todayilearned]
---

- 개인 공부 목적으로 작성한 포스팅입니다.
- 아래 출처를 참고하여 작성하였습니다. :)

### 상황

- 재배포 과정 중 AWS 인스턴스 상태 검사가 실패해서 로그를 확인해보니 `HikariPool - Thread starvation or clock leap detected`라는 메시지가 있었습니다.
- 이게 어떤 오류인지 확인한 내용을 정리합니다.
- 요약하자면, 해당 오류는 `consumer thread` 갯수에 따른 충분한 `HikariCP의 maximum pool size`를 설정하지 못해 Dead lock이 발생하면 나타나는 메시지입니다.

### Hikari Default maximumPoolSize(10개)

- https://github.com/brettwooldridge/HikariCP

### Spring Default min-spare Thread(10개)

- `server.tomcat.threads.min-spare` 속성은 Minimum amount of worker threads 의미로 `기본값은 10개`입니다.
- 즉, 스프링부트 서버의 `Worker thread` 갯수의 기본값은 10개입니다.

### poolSize = Tn x (Cm - 1) + 1

- `Tn`은 Worker Thread 갯수를 의미합니다.
- `Cm`은 쓰레드가 작업을 하면서 필요로 하는 `Connection` 갯수입니다.
  - 필요한 커낵션은 2개입니다.

### 필요한 커넥션 갯수: 2개

JPA를 사용하여 데이터를 처리하는 경우 필요한 커넥션 갯수는 2개입니다.

- `Service`의 메소드가 사용될 때 `Service의 트랜잭션이 시작`됩니다.
- 그리고 Service의 메소드에서 Repository를 사용할 때 `Repository의 트랜잭션이 시작`됩니다.
- 스프링 컨테이너는 `트랜잭션 범위의 영속성 컨텍스트 전략`을 사용한다고 합니다.
  - 즉, 요청을 처리하는 한 개의 Thread에서 Service의 트랜잭션과 Repository의 트랜잭션 때문에 `2개의 영속성 컨텍스트`가 생깁니다.
  - 그리고 2개의 영속성 컨텍스트를 위한 엔티티 매니저도 2개가 생겼으므로, `Connection`이 2개가 필요한 상황입니다.
- 그러나 충분한 `maximum pool size`를 제공하지 않아서, 남은 `Connection`이 없는 상황이 발생해서 쓰레드는 작업을 끝내지 못하고 데드락에 걸리는 상황이 발생한 것입니다.

### 결론: 수정사항

- 위 계산 공식에 맞게 `10 * (2-1) + 1`개 즉, 11개 이상의 커넥션 풀을 제공하도록 수정했습니다.

```yml
spring:
  datasource:
    hikari:
      maximum-pool-size: 20
```

---

- 출처
  - [인스턴스 상태검사 실패, Thread starvation or clock leap detected, Dead Lock, hikari 오류-1](https://velog.io/@mbsik6082/Thread-starvation-or-clock-leap-detected-Dead-Lock-hikari-%EC%98%A4%EB%A5%98)
  - [HikariCP Maximum Pool Size 설정 시, 고려해야할 부분](https://jaehun2841.github.io/2020/01/27/2020-01-27-hikaricp-maximum-pool-size-tuning/)
  - [Common Application Properties](https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html#appendix.application-properties.core)
  - [About Pool Sizing](https://github.com/brettwooldridge/HikariCP/wiki/About-Pool-Sizing#pool-locking)
