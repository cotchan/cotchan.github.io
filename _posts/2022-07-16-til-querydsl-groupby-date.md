---
layout: post
title: querydsl에서 date 기준으로 통계 쿼리 만들기 (feat. mysql)
subtitle: (groupBy) datetime 필드를 date 유형으로 묶기
categories: todayilearned
tags: [todayilearned]
---

- 개인 공부 목적으로 작성한 포스팅입니다.
- 아래 출처를 참고하여 작성하였습니다. :)

### 상황

- 개발 화면중에 아래와 같이 깃허브 잔디 심기처럼 `한 달 동안의 사용이력`을 데이터로 만들어야 했습니다.

![스크린샷 2022-07-16 오후 3 06 52](https://user-images.githubusercontent.com/75410527/179343407-8efc0a30-2c7d-47ec-837c-cd090fa81584.png)


#### date & datetime

- 도메인에서 사용하던 `datetime` 필드는 두 가지였습니다.
  - 생성시점을 나타내는 `created_at`
  - 수정시점을 나타내는 `updated_at`
- 그리고 위에서 필요한 쿼리는 `time`은 필요 없으므로 `date`를 기준으로 그룹핑 해줘야합니다.

### 해결방법

#### StringTemplate 정의

- `StringTemplate`을 사용하여 `datetime` 필드에서 `date`만 사용하도록 지정했습니다.

```java
private StringTemplate localDateFormat() {
    return Expressions
            .stringTemplate("DATE_FORMAT({0}, {1})", quizLog.createdAt, ConstantImpl.create("%Y-%m-%d"));
}
```

#### date 형식을 바인딩할 queryDto 정의

- 위에서 정의한 `localDateFormat()`을 기준으로 `groupBy`가 될 것이므로 해당 필드가 SELECT 절에 등장합니다.
- 단, `localDateFormat()`은 `String`입니다.
- 그래서 String이 아닌 `date` 의미에 맞게 `LocalDate`로 변환하도록 queryDto에서 형변환을 해줘야합니다.

```java
@Getter
public class QuizLogTraceQueryDto {
    private LocalDate date;
    private SolvedState solvedState;

    public QuizLogTraceQueryDto(String yyyymmdd, SolvedState solvedState) {
        this.date = LocalDate.parse(yyyymmdd, DateTimeFormatter.ofPattern("yyyy-MM-dd"));
        this.solvedState = solvedState;
    }
}
```

#### groupBy에 StringTemplate 적용

- `dateTime 필드`(created_at)를 `date` 기준으로 `groupBy`한 최종 코드입니다.

이 과정에서 중요한 점은 `3가지`입니다.

1. `localDateFormat()` 메서드를 별도로 정의하여 `.groupBy` 기준으로 넣기
2. groupBy 기준인 `localDateFormat()` 필드가 SELECT 절에 등장
3. `String` 필드인 `localDateFormat()` date 포맷에 맞게 `DTO` 클래스에서 형변환

```java
@RequiredArgsConstructor
@Repository
public class QuizLogQueryRepository {

    private final JPAQueryFactory jpaQueryFactory;

    public List<QuizLogTraceQueryDto> getQuizLogTraces(Long userId, LocalDateTime time) {
        return jpaQueryFactory
                .select(
                    Projections.constructor(QuizLogTraceQueryDto.class,
                        localDateFormat(),
                        quizLog.solvedState.max()
                    )
                )
                .from(quizLog)
                .where(
                        quizLog.user.id.eq(userId),
                        quizLog.createdAt.between(time.minusMonths(1), time)
                )
                .groupBy(localDateFormat())
                .orderBy(localDateFormat().desc()).fetch();
    }

    private StringTemplate localDateFormat() {
        return Expressions
                .stringTemplate("DATE_FORMAT({0}, {1})", quizLog.createdAt, ConstantImpl.create("%Y-%m-%d"));
    }
}
```

### 결과

- 위의 쿼리(+ 후처리)를 통해 얻은 `한 달 동안의 사용이력` 데이터입니다.

```json
[
  {
    "date": "2022-07-15",
    "state": 0
  },
  {
    "date": "2022-07-14",
    "state": 1
  },
  {
    "date": "2022-07-13",
    "state": 1
  },
  {
    "date": "2022-07-12",
    "state": 0
  },
  {
    "date": "2022-07-11",
    "state": 1
  },
  {
    "date": "2022-07-01",
    "state": 1
  },
  {
    "date": "2022-06-30",
    "state": 0
  },
  {
    "date": "2022-06-29",
    "state": 0
  },
  {
    "date": "2022-06-28",
    "state": 0
  },
  {
    "date": "2022-06-27",
    "state": 0
  },
  {
    "date": "2022-06-26",
    "state": 0
  },
  {
    "date": "2022-06-20",
    "state": 0
  },
  {
    "date": "2022-06-19",
    "state": 0
  },
  {
    "date": "2022-06-18",
    "state": 0
  },
  {
    "date": "2022-06-17",
    "state": 0
  },
  {
    "date": "2022-06-16",
    "state": 0
  }
]
```

---

- 출처
  - [[querydsl, mysql] DATE_FORMAT 등 이용해 groupby 사용하기](https://lemontia.tistory.com/929)
  - [MySQL datetime 월별 GROUP BY](https://zetawiki.com/wiki/MySQL_datetime_%EC%9B%94%EB%B3%84_GROUP_BY)
  - [[QueryDSL] Date 별 Group By 는 Expressions.dateTemplate !! 혹은 StringTemplate ??](https://kdhyo98.tistory.com/20)
  - [MySQL - 일별통계, 주간통계, 월간통계](https://bluexmas.tistory.com/626)
  - [MySQL CASE WHEN GROUP BY(원하는 데이타끼리 그룹핑 하기)](https://zzarungna.com/1419)
