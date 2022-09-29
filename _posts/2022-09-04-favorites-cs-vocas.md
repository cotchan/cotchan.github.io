---
layout: post
title: CS 단어 정리
subtitle: 들을 때 마다 헷갈리는 단어들 정리
categories: favorites
tags: [favorites]
---

- 개인 공부 목적으로 작성한 포스팅입니다.
- 아래 출처를 참고하여 작성하였습니다. :)
- 계속 업데이트할 예정입니다.

### I/O(input/output)

- I/O란, `cpu와 메모리를 제외한` 나머지 시스템 리소스에 접근하는 것을 의미합니다.
- 데이터의 입출력을 의미합니다.
  - 파일을 읽고 쓰기
  - 네트워크의 어딘가와 `(소켓을 통해)` 데이터를 주고받는 것
  - 입출력 장치와 데이터를 주고 받는 것
- 공통적으로 I/O는 `user-space aplication`에서 `직접 수행할 수 없습니다.`
- 즉, I/O를 수행하기 위해서는 `커널에 한 번 이상 시스템 콜을 보내야`합니다.

#### I/O 종류

- network(socket) i/o
- file i/o
- pipe i/o(프로세스 간 통신)
- device i/o

### TPS

- TPS(티피에스)란 `Transaction Per Second`의 약자로서, `1초당 처리할 수 있는 트랜잭션의 개수를 의미`합니다.
- `100만 TPS는 1초당 100만 건`의 `트랜잭션을 처리할 수 있는 속도`를 말합니다.

### Load Average

- Load Average란, 프로세스의 여러 상태 중 `R` 또는 `D` 상태에 있는 프로세스들의 개수를 1분, 5분, 15분 단위로 평균을 낸 값
- 즉, `얼마나 많은 프로세스`가 `실행 중 혹은 실행 대기중인지`를 의미하는 수치
- 시스템 운영시 권장하는 에버리지는 `70%`인 `0.7` 이하 이며 그 이상일 경우 시스템에 이상이 없는지 반드시 체크 해야합니다.

<img width="480" alt="스크린샷 2022-09-04 오후 3 26 53" src="https://user-images.githubusercontent.com/75410527/188300768-7501e809-f948-4e22-997c-189b0ab83d73.png">

- 출처: scoutapp
- 퍼온 곳: [1. Load Average에 관해서](https://brunch.co.kr/@dreaminz/1)

#### Load Average가 높다?

- `Load Average가 높다`는 뜻은, 많은 프로세스가 현재 실행 중이거나, 네트워크 또는 디스크 작업 처리를 위해 대기 상태에 있다는 것을 말합니다.

#### '실행' 상태란

- 프로세스의 `R(Running)` 상태를 의미합니다.
- 프로세스가 `실행`중인 상태는 CPU 자원을 소모하고 있는 실행 중인 상태를 의미합니다.
- R이 많다는 것은 CPU 리소스 측면에서 부하가 많다는 뜻입니다.

#### '실행 대기중' 상태란

- 프로세스의 `D(Uninterruptible waiting)` 상태를 의미합니다.
- 프로세스가 `실행 대기중`인 상태는 I/O에 대해 대기하는 상태로, 다른 어떤 일도 할 수 없는 상태를 의미합니다.
- 보통 `디스크 또는 네트워크 등 I/O 작업 처리를 기다리는` 프로세스의 상태를 말합니다.

### Throughput

- `컴퓨터 시스템의 처리 능력`을 나타내는 단어로, `단위 시간당 처리할 수 있는 작업양`을 의미합니다.
- Throughput is a measure of how many units of information `a system can process` in `a given amount of time.`

### Backpressure

- 소프트웨어 시스템에서 백프레셔는 `트래픽 통신에 과부하를 일으키는 기능을 의미`합니다.
- 다시 말해, 데이터를 보내는 쪽에서 너무 많은 데이터를 계속 Push 함으로써 데이터 소비자가 처리할 수 없는 상태를 의미합니다.
- 리액티브 프로그래밍의 `non-blocking` 특성으로 인해 서버는 전체 스트림을 한 번에 보내지 않고, 데이터가 사용 가능한 즉시 데이터를 푸시할 수 있습니다. 따라서 클라이언트는 이벤트를 수신하고 처리하는 데 더 적은 시간을 기다립니다. 그러나 위와 같은 `Backpressure` 문제가 발생할 수 있습니다.

### Payload

---

- 참고
  - [TPS - 해시넷 위키](http://wiki.hash.kr/index.php/TPS)
  - [1. Load Average에 관해서](https://brunch.co.kr/@dreaminz/1)
  - [[Linux]Load Average 란?](https://kim-dragon.tistory.com/45)
  - [쓰루풋(Throughput)이란?](https://m.blog.naver.com/sung_mk1919/221212568779)
  - [throughput](https://www.techtarget.com/searchnetworking/definition/throughput)
  - [Backpressure Mechanism in Spring WebFlux](https://www.baeldung.com/spring-webflux-backpressure)
  - [block I/O vs non-block I/O 개념을 설명합니다! 소켓 I/O를 예제로 주로 설명해요! I/O multiplexing(다중 입출력) 설명도 빠질 수 없겠죠? ;)](https://www.youtube.com/watch?v=mb-QHxVfmcs&ab_channel=%EC%89%AC%EC%9A%B4%EC%BD%94%EB%93%9C)
  - [cpu bound, io bound 의미를 설명합니다! 이에 따른 스레드 개수를 정하는 팁도 알려드립니다!](https://www.youtube.com/watch?v=qnVKEwjG_gM&t=77s&ab_channel=%EC%89%AC%EC%9A%B4%EC%BD%94%EB%93%9C)
