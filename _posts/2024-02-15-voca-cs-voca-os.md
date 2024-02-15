---
layout: post
title: CS 단어 정리(운영체제)
subtitle: Computer Science 단어 정리
categories: voca
tags: [voca]
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

> 헤더가 실어나르는 대상을 Payload라고 함(택배 내용물에 해당)

- `단위 데이터`라고 하는 것(ex. 패킷, 프레임 등)은 `네트워크에서 항상 2개로 쪼갬`
  - 앞단은 `Header`, 헤더가 실어나르는 대상을 `Payload`라고 함(택배 내용물에 해당)

### 레지스터

- 레지스터는 `CPU에 존재`하는 다목적 저장 공간
  - CPU 레지스터는 각종 명령어들을 수행하기 위해 여러 데이터를 저장하는 존재
  - CPU가 요청을 처리하는데 필요한 데이터를 일시적으로 저장하는 다목적 공간
  - 현재 계산을 수행중인 값을 저장하는 데 사용됨.
- 레지스터는 `데이터와 명령어를 저장하는 역할`을 함.
- 레지스터는 컴퓨터의 프로세서 내에서 자료를 보관하는 아주 빠른 기억 장소
- 대부분의 현대 프로세서는 메인 메모리에서 레지스터로 데이터를 옮겨와 데이터를 처리한 후 그 내용을 다시 레지스터에서 메인 메모리로 저장하는 `로드-스토어 설계`를 사용하
  - 연산을 위한 데이터를 레지스터에 저장하고, 그 결과값도 레지스터에 저장
- 레지스터는 메모리 계층의 최상위에 위치하며, `가장 빠른 속도로 접근 가능한 메모리`
- 속도가 엄청 빠르다. `레지스터` > `메모리` > `하드디스크` 순서

<img width="500" alt="스크린샷 2022-09-30 오후 6 40 26" src="https://user-images.githubusercontent.com/75410527/193243059-9cbdcebe-6d6b-444b-92ed-66acafc4b03d.png">

#### 레지스터의 종류

- 데이터 레지스터 : 정수 값을 저장할 수 있는 레지스터.
- 주소 레지스터 : 메모리 주소를 저장하여 메모리 접근에 사용되는 레지스터. 어떤 프로세서에서는, 주소를 저장하는 것이 아니라 조작하기 위한 목적으로 색인 레지스터를 사용하기도 함.
- 범용 레지스터 : 데이터와 주소를 모두 저장할 수 있는 레지스터.
- 부동 소수점 레지스터 : 많은 시스템에서 부동소수점 값을 저장하기 위해 사용된다.
- 상수 레지스터 : 0이나 1 등 고정된 값을 저장하고 있는 레지스터.
- 특수 레지스터 : `프로그램의 상태를 저장`한다. `프로그램 카운터`, `스택 포인터`, 상태 레지스터 등이 있다.
- 명령 레지스터 : 현재 실행중인 명령어를 저장한다.
- 색인 레지스터 : 실행중에 피연산자의 주소를 계산하는 데 사용된다.

#### Cache vs 레지스터

- 캐시는 `cpu와 별도로 있는 공간`이며, 메인 메모리와 cpu 간의 `속도 차이를 극복`하기 위한 것
- 레지스터는 `cpu 안에서 연산을 처리하기 위하여 데이터를 저장하는 공간`

### Context

- `프로세스/스레드의 상태`
- 여기서 말하는 `상태`라는 건, `CPU에서의 상태`, 혹은 `메모리에서의 상태`를 의미한다.

#### Context Swtiching

- CPU에서 수행되기 위해서 어느 한 프로세스에서 다른 프로세스로 교체되는 것
- CPU/코어에서 실행 중이던 프로세스/스레드가 다른 프로세스/스레드로 교체되는 것

> 커널 모드에서 실행된다.  
> CPU의 레지스터 상태가 교체된다.

### 커널모드

- 커널 모드란, 프로세스가 실행되다가 하드웨어와 밀접한 일들 혹은 컴퓨터의 리소스들을 다뤄야하는 상황이 오면 이 때는 프로세스가 직접 컴퓨터의 리소스에 접근하는 게 아니라, 운영체제를 통해 접근합니다.
- 특히 운영체제 중에서도 커널을 통해 접근합니다.
- 이 때 `프로세스에서 커널로 통제권이 넘어가서` `커널에 의해 실행`되는 걸 커널모드라고 합니다.

### CPU atomic

`CPU atomic`한 연산이란,

- 실행 중간에 간섭받거나 중단되지 않는다.
- 같은 메모리 영역에 대해 동시에 실행되지 않는다.
  - 같은 메모리영역에 대해 실행되지 않는다는 건,
  - 동일한 parameter에 대해 (예시. `lock`) 두 개 이상의 스레드나 프로세스가 동시에 호출한다고 해도
  - `CPU 레벨에서 알아서` 먼저 하나를 실행시키고, 하나가 실행이 끝나고 이어서 다른 하나를 실행시킴.
  - `이렇게 동기화해서 실행시킨다는 뜻`

### race condition

- 여러 프로세스/스레드가 `동시에 같은 데이터를 조작`할 때
- `타이밍`이나 `접근 순서`에 따라 `결과가 달라질 수 있는 상황`

### 동기화(synchronization)

- 여러 프로세스/스레드를 동시에 실행해도 `공유 데이터의 일관성을 유지`하는 것
- `한 순간에` `하나의 자원`을 `하나의 스레드/프로세스가 사용하도록 보장/제어`하는 것

### Thread Safe

- 어떤 함수나 변수, 객체가 `여러 스레드로부터 동시에 접근이 이루어져도` 프로그램 실행에 문제가 없음을 의미

> 멀티 스레드 프로그래밍에서 사용하는 용어

### critical section

- `공유 데이터의 일관성을 보장하기 위해` `하나의 프로세스/스레드만 진입해서 실행 가능`한 영역

#### mutual exclusion

- `하나의 프로세스/스레드만 진입`해서 실행하게 하는 것

### mutex

> 핵심은 LOCK을 가질 수 있을 때 까지 휴식

- critical section에서 `mutual exclusion을 보장`하는 장치
- critical section에 진입하려면 `mutex lock`을 획득해야 함
- mutex lock을 획득하지 못한 스레드는 `큐에 들어간 후 대기(waiting) 상태로 전환`
  - ex.entry queue
- mutex lock을 쥔 스레드가 lock을 반환하면, lock을 기다리며 큐에 대기 상태로 있던 스레드 중 하나가 실행

### semaphore

- 하나 이상의 프로세스/스레드가 critical section에 접근 가능하도록 하는 장치
  - 세마포어의 특별한 기능이라 할 수 있는 `시그널 메커니즘`을 통해 제어한다.
- `시그널 메커니즘`을 통해 프로세스/스레드 간 `동기화(작업 순서)를 보장`해줄 수 있다.

> lock이라는 이름이 wait  
> unlock이라는 이름이 signal

```java
semaphore->wait();
... critical section
semaphore->signal();
```

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
  - [컨텍스트 스위칭 뽀개기! 의미와 종류와 왜 스레드 컨텍스트 스위칭이 더 빠르다고 하는지까지..! 이 모든 것을 시원~~하게 설명합니다!!](https://youtu.be/Xh9Nt7y07FE)
  - [레지스터 (Register)](https://plummmm.tistory.com/113)
  - [캐시와 레지스터의 차이가 무엇일까요???](https://melonicedlatte.com/computer/2018/11/07/190754.html)
  - [동기화(synchronization), 경쟁 조건(race condition), 임계 영역(critical section)을 자세하게 설명합니다! 헷갈리시는 분들 꼭 보세요!](https://www.youtube.com/watch?v=vp0Gckz3z64&ab_channel=%EC%89%AC%EC%9A%B4%EC%BD%94%EB%93%9C)
  - [스핀락(spinlock) 뮤텍스(mutex) 세마포(semaphore) 각각의 특징과 차이 완벽 설명! 뮤텍스는 바이너리 세마포가 아니라는 것도 설명합니다!](https://www.youtube.com/watch?v=gTkvX2Awj6g&t=594s&ab_channel=%EC%89%AC%EC%9A%B4%EC%BD%94%EB%93%9C)
  - [모니터가 어떻게 동기화에 사용되는지 아주 자세히 설명합니다! 자바에서 모니터는 어떤 모습인지도 설명하니 헷갈리시는 분들 꼭 보세요!](https://www.youtube.com/watch?v=Dms1oBmRAlo&ab_channel=%EC%89%AC%EC%9A%B4%EC%BD%94%EB%93%9C)
  - [널널한 개발자 TV](https://www.youtube.com/channel/UCdGTtaI-ERLjzZNLuBj3X6A)
