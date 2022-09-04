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

### TPS

- TPS(티피에스)란 `Transaction per Second`의 약자로서, `1초당 처리할 수 있는 트랜잭션의 개수를 의미`합니다.
- 100만 TPS는 1초당 100만 건의 트랜잭션을 처리할 수 있는 속도를 말합니다.

### Load Average

- Load Average란, 프로세스의 여러 상태 중 `R` 또는 `D` 상태에 있는 프로세스들의 개수를 1분, 5분, 15분 단위로 평균을 낸 값
- 즉, `얼마나 많은 프로세스`가 `실행 중 혹은 실행 대기중인지`를 의미하는 수치
- 시스템 운영시 권장하는 에버리지는 `70%`인 `0.7` 이하 이며 그 이상일 경우 시스템에 이상이 없는지 반드시 체크 해야합니다.

#### Load Average가 높다?

- `Load Average가 높다`는 뜻은, 많은 프로세스가 현재 실행 중이거나, 네트워크 또는 디스크 작업 처리를 위해 대기 상태에 있다는 것을 말합니다.

#### 실행

- 프로세스의 `R(Running)` 상태를 의미합니다.
- 프로세스가 `실행`중인 상태는 CPU 자원을 소모하고 있는 실행 중인 상태를 의미합니다.
- R이 많다는 것은 CPU 리소스 측면에서 부하가 많다는 뜻입니다.

#### 실행 대기중

- 프로세스의 `D(Uninterruptible waiting)` 상태를 의미합니다.
- 프로세스가 `실행 대기중`인 상태는 I/O에 대해 대기하는 상태로, 다른 어떤 일도 할 수 없는 상태를 의미합니다.
- 보통 `디스크 또는 네트워크 등 I/O 작업 처리를 기다리는` 프로세스의 상태를 말합니다.

---

- 참고
  - [TPS - 해시넷 위키](http://wiki.hash.kr/index.php/TPS)
  - [1. Load Average에 관해서](https://brunch.co.kr/@dreaminz/1)
  - [[Linux]Load Average 란?](https://kim-dragon.tistory.com/45)
  
