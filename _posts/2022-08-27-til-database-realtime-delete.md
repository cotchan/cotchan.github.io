---
layout: post
title: TIL 작성 중
subtitle: 운영 DB에서 DELETE문을 지양해야하는 이유
categories: todayilearned
tags: [todayilearned]
---

- 개인 공부 목적으로 작성한 포스팅입니다.
- 아직 작성 중입니다.
- 아래 출처를 참고하여 작성하였습니다. :)

### DELETE 문 지양 이유

- `auto_increment id`가 들쑥날쑥 해지는 것을 방지
- 혹시 모를 버그나 보안위험이 생겨도 데이터를 지키기 위해
- `데이터 추적을 위해` 따로 로그를 남겨둔다거나 state 값을 이용해서 update 문으로 처리

---

- 출처
  - [delete문 한번더 질문하겠습니다...](https://www.sqler.com/board_SQLQA/321227)
