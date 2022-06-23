---
layout: post
title: DataSource에 대한 이해
subtitle: JDBC(Java Database Connectivity)
categories: todayilearned
tags: [todayilearned]
---

- 개인 공부 목적으로 작성한 포스팅입니다.
- 아래 출처를 참고하여 작성하였습니다. :)

---

### DataSource 탄생 배경

- DB 커넥션을 얻는 방법은 `JDBC DriverManager`를 직접 사용하거나, 커넥션 풀을 사용하는 등 다양한 방법이 존재합니다.
- 그러나 커넥션을 얻는 방법이 바뀌는 경우 `커넥션을 획득하는 애플리케이션 코드도 함께 변경`해야 합니다.
  - 의존성이 바뀌기 때문입니다.

### DataSource란

DataSource는 `커넥션을 획득하는 방법을 추상화한 것`입니다.

- 자바에서는 위와 같은 문제를 해결하기 위해 `javax.sql.DataSource`라는 인터페이스를 제공합니다.
- DataSource는 커넥션을 획득하는 방법을 추상화하는 인터페이스입니다.
- 이 인터페이스의 핵심 기능은 `커넥션 조회` 한 가지입니다.

### 핵심 기능

- DB 커넥션을 어떻게 획득할거냐?
- 구현은 내가 DriverManager를 통해 하거나 Connection Pool을 사용해서 하거나 상관없음
- 그냥 '커넥션 주세요' 이 방법만 추상화해놓은 것

```java
public interface DataSource {
    Connection getConnection() throws SQLException;
}
```

---

- 참고
  - [김영한, 스프링 DB 1편 - 데이터 접근 핵심 원리](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-db-1/dashboard)
