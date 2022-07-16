---
layout: post
title: Fetch Join과 일반 Join(with DTO)
subtitle: 언제 Fetch Join을 사용하고, 언제 일반 Join을 사용?
categories: todayilearned
tags: [todayilearned]
---

- 개인 공부 목적으로 작성한 포스팅입니다.
- 아래 출처를 참고하여 작성하였습니다. :)

### 테스트

- Querydsl을 사용해서 `DTO를 조회`하는 코드입니다.

```java
@RequiredArgsConstructor
@Repository
public class QuizQueryRepository {

    private final JPAQueryFactory jpaQueryFactory;

    public List<QuizQueryDto> findAll(Long userId, Pageable pageable) {
        return jpaQueryFactory.select(
                    Projections.constructor(QuizQueryDto.class,
                        quizSolvedState.id,
                        quiz.id,
                        quiz.title,
                        quiz.quizUrl,
                        quiz.level,
                        quiz.platform
                    )
                )
                .from(quizSolvedState)
                    .join(quizSolvedState.user, user)
                        .on(quizSolvedState.user.id.eq(userId))
                    .rightJoin(quizSolvedState.quiz, quiz)
                .where(
                    quizSolvedState.solvedState.eq(SolvedState.NOT_PICKED)
                    .or(quizSolvedState.id.isNull())
                )
                .orderBy(quizSolvedState.updatedAt.desc())
                    .limit(pageable.getPageSize())
                    .offset(pageable.getOffset())
                .fetch();
    }
}


@Getter
@AllArgsConstructor
public class QuizQueryDto {
    private Long quizSolvedStateId;
    private Long quizId;
    private String title;
    private String quizUrl;
    private QuizLevel quizLevel;
    private QuizPlatform quizPlatform;
}
```

#### 결과

- 쿼리에서 `quiz`, `user`에 접근했지만 쿼리가 한 번만 나간 것을 확인할 수 있습니다.

```sql
Hibernate:
    select
        quizsolved0_.id as col_0_0_,
        quiz2_.id as col_1_0_,
        quiz2_.title as col_2_0_,
        quiz2_.quiz_url as col_3_0_,
        quiz2_.level as col_4_0_,
        quiz2_.platform as col_5_0_
    from
        quiz_solved_states quizsolved0_
    inner join
        users user1_
            on quizsolved0_.user_id=user1_.id
            and (
                quizsolved0_.user_id=?
            )
    right outer join
        quizzes quiz2_
            on quizsolved0_.quiz_id=quiz2_.id
    where
        quizsolved0_.solved_state=?
        or quizsolved0_.id is null
    order by
        quizsolved0_.updated_at desc limit ?
```

### 결론

- `DTO로 조회할 때는` 일반 Join을 사용해도 `N+1이 발생하지 않습니다.`

#### 추신

- 그리고 fetch join 사용 시 select 필드에 연관관계의 주인 엔티티가 나오지 않으면 런타임 에러가 나옵니다.
- 그러므로 DTO 조회할 때 fetch join을 걸고 엔티티를 사용하지 않으면 에러가 나오는 경험을 할 수 있습니다.

```
query specified join fetching, but the owner of the fetched association was not present in the select list
```

### 사용해야 할 때

#### Fetch Join

- `엔티티 조회 시`, 하나 이상의 엔티티를 `한 번에 영속화해야 할 때`
  - e.g. 두 엔티티의 영속화가 필요로 할 때

뻔한 내용일 수 있지만, 중요한 건 `엔티티 조회 시` Fetch Join을 고려해야 합니다.

#### 일반 Join(with DTO)

- DTO 조회 시(`엔티티 조회가 필요없는 경우`)
- 영속화가 필요없는 엔티티 조회

DTO로 조회한다면 일반 Join을 사용하면 됩니다.

---

- 출처
  - [Fetch Join vs 일반 Join(feat.DTO)](https://velog.io/@heoseungyeon/Fetch-Join-vs-%EC%9D%BC%EB%B0%98-Joinfeat.DTO)
