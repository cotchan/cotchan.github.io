---
layout: post
title: Top-down DP로 풀이전환 시 고려해야 할 것들
subtitle: dp
categories: algorithm
tags: [algorithm]
---

- 개인 공부 목적으로 작성한 포스팅입니다.
- 아래 출처를 참고하여 작성하였습니다. :)

---

### intro

- [외판원 순회](https://www.acmicpc.net/problem/2098) 문제를 비트마스킹만 사용하면 시간 복잡도가 초과합니다.
- 같은 상태지만 서로 다른 순서로 접근하는 경우의 수를 모두 중복 연산하기 때문입니다.
  - 13425 순서로 방문 -> 결국 방문상태는 12345
  - 15432 순서로 방문 -> 결국 방문상태는 12345
- 위 문제를 Topdown dp로 바꾸면서 중복 연산을 제거하려고 할 때 중요한 부분에 대해 적어봅니다.

### dp 배열 초기값과 방문 표시 상태 분리하기

- dp 배열을 초기화할 때 값과, `INF`나 `IMPOSSIBLE` 같이 올바르지 않은 정답을 식별하는 값은 달라야 합니다.

#### 올바른 예

- dp 배열 초기화는 `-1`로, 올바르지 않은 정답 식별은 `IMPOSSIBLE`로

```java
public static final int IMPOSSIBLE = 100_000_000;

public static void main(String[] args) {
    int[][] dp = new int[N][1<<N];

    for (int i = 0; i < N; ++i) {
        Arrays.fill(dp[i], -1);
    }
}

public static int solve(int nowNode, int state) {
    if (dp[nowNode][state] != -1) {
        return dp[nowNode][state];
    }

    //dp[nowNode][state]를 호출했다는 표시
    dp[nowNode][state] = IMPOSSIBLE;

    //...

    return dp[nowNode][state];
}
```

### dp 점화식은 caller가 callee를 결과값을 사용하는 상태로 정의

- dp 목적은 메모이제이션을 통해서 `겹치는 상태를 딱 한 번만 연산하는 것`입니다.
- 겹치는 상태를 한 번만 연산해도 된다는 건 이전 상태의 결과값을 사용해서 다음 상태를 구할 수 있기 때문입니다.

위의 `외판원 순회` 문제 해결을 위한 점화식은 아래처럼 정의할 수 있습니다.

```
dp[i][j] = 현재 있는 도시가 i이고 이미 방문한 도시들의 집합이 j일때,
방문하지 않은 나머지 도시들을 모두 방문한 뒤 출발 도시로 돌아올 때 드는 최소 비용.
```

- 위 점화식이 올바른 점화식인 이유는, 현재 도시에서 다음 도시를 방문하면서 i값과 j값이 바뀔 때 caller가 callee의 결과값을 사용하는 형태로 정의되어 있기 때문입니다.
- 즉 i값과 j값이 바뀌면서 추가적인 함수 호출이 일어나는데, 추가적인 함수 호출의 결과값을 caller가 사용해서 자신의 결과값에 반영합니다.

### 매개변수로 저장했던 값은 dp 배열에 저장

dp를 사용하지 않고 중복 연산을 사용했을 때

- parameter로 결과값 `int cost`를 가지고 다니면서 `ans` 최솟값을 갱신하는 것을 볼 수 있습니다.

```java
public static void solve(int nowNode, int state, int cost) {
    //모든 노드를 방문했을 때
    if (state == ((1 << N) - 1)) {
        if (W[nowNode][0] != 0) {
            ans = Math.min(ans, cost + W[nowNode][0]);
        }

        return;
    }

    for (int i = 0; i < N; ++i) {
        //방문안했고, 이동할 수 있을 때 이동
        if (W[nowNode][i] == 0 || ((state & (1 << i)) != 0)) continue;

        int nxtState = (state | (1 << i));
        int nxtCost = cost + W[nowNode][i];
        solve(i, nxtState, nxtCost);
    }
}
```

Topdown dp 적용시

- 위에서 사용했던 파라미터 `int cost`가 결과값을 `dp` 배열에 저장하는 것을 볼 수 있습니다.

```java
public static int solve(int nowNode, int state) {

    //모든 노드를 방문했을 때
    if (state == ((1 << N) - 1)) {
        return W[nowNode][0] == 0 ? IMPOSSIBLE : W[nowNode][0];
    }

    if (dp[nowNode][state] != -1) {
        return dp[nowNode][state];
    }

    //dp[nowNode][state]를 호출했다는 표시
    dp[nowNode][state] = IMPOSSIBLE;

    for (int i = 0; i < N; ++i) {
        //방문안했고, 이동할 수 있을 때
        if (W[nowNode][i] == 0 || ((state & (1 << i)) != 0)) continue;

        int nxtState = (state | (1 << i));
        dp[nowNode][state] = Math.min(dp[nowNode][state], solve(i, nxtState) + W[nowNode][i]);
    }

    return dp[nowNode][state];
}
```

---

- 참고
  - [[BOJ] 백준 2098번 외판원 순회 (Java)](https://loosie.tistory.com/271#%ED%92%80%EC%9D%B4_%EC%BD%94%EB%93%9C)
  - [(JAVA) 백준 2098번 : 외판원 순회 --- [DP, TSP, 비트마스크]](https://maivve.tistory.com/306)
