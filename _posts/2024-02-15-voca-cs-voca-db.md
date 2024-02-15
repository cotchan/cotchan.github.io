---
layout: post
title: CS 단어 정리(데이터베이스)
subtitle: Computer Science 단어 정리
categories: voca
tags: [voca]
---

- 개인 공부 목적으로 작성한 포스팅입니다.
- 아래 출처를 참고하여 작성하였습니다. :)
- 계속 업데이트할 예정입니다.

### JOIN

#### Equi Join

- 등가조인[Equi Join]

```sql
SELECT * FROM 테이블1, 테이블2
	WHERE 테이블명1.컬럼명1=테이블명2.컬럼명2;
```

- 테이블간의 `공통 컬럼을 활용`하여 각 테이블의 특정 컬럼에 `일치한 데이터를 기준으로 연결`하는 방법
  - 두 개의 테이블 간의 일치하는 것을 조인(교집합)
- `등가조인(Equi Join)` = `내부조인(Inner Join)` = `단순조인(Simple Join)`

#### NATURAL JOIN

```sql
SELECT 출력할 칼럼명1, 칼럼명2,..
	FROM 테이블명1 NATURAL JOIN 테이블명2;
```

- 두 테이블 간 `동일한 이름을 갖는 모든 칼럼`에 대해 `EQUI JOIN을 수행`하는 방법
- USING 조건절, ON조건절, WHERE절에서의 JOIN조건을 함께 정의 불가
- NATURAL JOIN에 사용된 칼럼들은 `같은 데이터 유형`이어야 함
- NATURAL JOIN은 Alias(별칭)이나 테이블명과 같은 접두사(ex> EMP.DEPTNO)를 붙일 수 없음

---

- 참고
  - [[개미의 걸음 SQLD 2과목] JOIN(외부조인, 내부조인, 등가조인 , 비등가조인 , 셀프조인, 네츄럴조인, 크로스 조인)](https://2030bigdata.tistory.com/222)
