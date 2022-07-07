---
layout: post
title: 구간합 구하기
subtitle: 구간합을 구하는 여러가지 기법
categories: ps
tags: [ps]
---

- 개인 공부 목적으로 작성한 포스팅입니다.
- 아래 출처를 참고하여 작성하였습니다. :)
- 계속 업데이트할 예정입니다.

### 구간합(feat. 원형 구간)

예시 문제: [피자판매](https://www.acmicpc.net/problem/2632)

- `i~j까지의 구간`에 대한 구간합을 prefixSum 배열을 쓰지 않고 단순하게 구한다면 `O(N^2)`이 필요합니다.
  - 중요한 건 `st`, `len` 형태의 `2중 포문으로 정의`할 수 있습니다.
    - `st`: 구간합 계산을 시작하는 인덱스
    - `len`: `st`부터 몇 개의 원소를 셀 건지 나타내는 길이
      - 범위가 `len < N-1`인 이유는 전체 범위 구간합은 `딱 한 번만` 구하면 되므로 `len < N`으로 하면 중복 계산이 되기 때문입니다.

```java
//원형 구간합 구하기

//sizeA는 pizzaA 배열 내 모든 구간합으로 만들 수 있는 '숫자'를 인덱스로 하고, 만들 수 있는 '경우의 수'를 배열값으로 합니다.
static int[] pizzaA, sizeA;

pizzaA = new int[N];
sizeA = new int[1000001];

for (int st = 0; st < N; ++st) {
    int sum = 0;
    for (int len = 0; len < N-1; ++len) {
        sum += pizzaA[(st+len) % N];    //원형 구간이므로 % 연산 적용
        sizeA[sum]++;   //len =0 일 때, len=1 일 때, len=2일 때, 다 저장하려고 이 때 ++ 연산
    }
}
```

---

- 참고
  - [[BOJ]2632: 피자판매](https://travelbeeee.tistory.com/489)
