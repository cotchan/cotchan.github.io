---
layout: post
title: Tree DP
subtitle: Tree DP
categories: algorithm
tags: [algorithm]
---

- 개인 공부 목적으로 작성한 포스팅입니다.
- 아래 출처를 참고하여 작성하였습니다. :)

### 대표 예시 문제

Tree DP 유형 문제입니다.

- [사회망 서비스(SNS)](https://www.acmicpc.net/problem/2533)
- [스크루지 민호 2](https://www.acmicpc.net/problem/12978)
- [우수 마을](https://www.acmicpc.net/problem/1949)

### Tree DP란

대표적인 Tree DP 유형 문제이라면 아래 두 가지 조건을 가지고 있습니다.

1. 주어진 자료구조는 `트리`
1. 부모 노드 값을 구하려면 자식 노드의 값을 통해서 구할 수 있을 때

1번 조건은 트리에 해당되는 특징이고, 2번 조건은 DP에 해당하는 특징입니다.

### 문제 해결 과정1

먼저 문제를 해결하기 위한 DP 테이블 정의와 점화식에 대해 알아보겠습니다.

1. DP 테이블 정의
   - 먼저 Tree DP는 `2차원 DP`로 트리의 상태를 정의합니다.
   - `dp[노드의 갯수][상태의 갯수]`
1. 점화식 정의
   - 문제마다 요구하는 점화식은 다르겠지만 대략 점화식의 큰 형태는 다음과 같습니다.
   - `dp[부모 노드][부모의 상태]` += `dp[자식 노드][자식의 상태]`
   - 즉, 부모 <---- 자식 방향으로 값을 업데이트 후 결과는 부모 노드의 상태 비교를 통해 알 수 있습니다.
   - `int ans = max(dp[root][0], dp[root][1])`

### 문제 해결 과정2

보통 Tree DP는 DP 테이블과 점화식을 가지고 `Top-down 형태의 DP` + `DFS`로 해결합니다.

1. 기저 사례(초기값) 초기화
   - DP의 특징입니다. 각 노드의 해당하는 상태는 단 한번만 호출됩니다.
   - 그러므로 가장 먼저 자신의 기저사례(초기값)를 초기화합니다.
1. 자식 먼저 모두 탐색
   - Tree DP는 자식 노드 정보가 모두 업데이트된 상태에서 부모 노드 값을 구할 수 있습니다.
   - 그러므로 자신의 자식 노드를 먼저 전부 호출해서 업데이트해줍니다.
1. 부모 정보 갱신
   - 2번 과정을 통해서 모두 갱신한 자식 정보를 통해 부모 노드 값을 구합니다.
   - 이 부분에서 각 문제에서 요구하는 부모/자식의 상태에 따른 점화식이 사용됩니다.

```java
// 예시
public static void dfs(int node) {

    // 1st. 기저 사례 셋팅
    dp[node][NORMAL] = 0;
    dp[node][EARLY] = 1;

    for (int nxt : trees[node]) {
        if (visit[nxt]) continue;

        visit[nxt] = true;
        dfs(nxt);   // 2nd. 자식먼저 탐색

        // 3rd. 부모 정보 갱신
        dp[node][NORMAL] += dp[nxt][EARLY];
        dp[node][EARLY] += Math.min(dp[nxt][NORMAL], dp[nxt][EARLY]);
    }
}
```

### [사회망 서비스(SNS)](https://www.acmicpc.net/problem/2533) 정답 코드

- 끝으로, 위에서 Tree DP 유형 문제라고 했던 문제의 정답 코드를 참고용으로 추가해놓겠습니다.

```java
import java.io.*;
import java.util.*;


public class Main {

    public static final int NORMAL = 0;
    public static final int EARLY = 1;
    public static int N;
    public static boolean[] visit;
    public static int[][] dp;
    public static List<Integer>[] trees;

    public static void main(String[] args) throws IOException {

        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));

        N = Integer.parseInt(br.readLine());

        dp = new int[N][2];
        visit = new boolean[N];
        trees = new List[N];
        for (int i = 0; i < N; ++i) {
            trees[i] = new ArrayList<>();
        }

        for (int i = 0; i < N-1; ++i) {
            String[] edgeInfo = br.readLine().split(" ");
            int src = Integer.parseInt(edgeInfo[0]);
            int dst = Integer.parseInt(edgeInfo[1]);
            src--; dst--;

            trees[src].add(dst);
            trees[dst].add(src);
        }

        visit[0] = true;
        dfs(0);

        System.out.println(Math.min(dp[0][NORMAL], dp[0][EARLY]));
    }

    public static void dfs(int node) {

        // 1st. 기저 사례 셋팅
        dp[node][NORMAL] = 0;
        dp[node][EARLY] = 1;

        for (int nxt : trees[node]) {
            if (visit[nxt]) continue;

            visit[nxt] = true;
            dfs(nxt);   // 2nd. 자식먼저 탐색

            // 3rd. 부모 정보 갱신
            dp[node][NORMAL] += dp[nxt][EARLY];
            dp[node][EARLY] += Math.min(dp[nxt][NORMAL], dp[nxt][EARLY]);
        }
    }
}
```

---

- 참고
  - [트리 dp : 스크루지 민호 문제로 간단한 것만 풀어 봅시다.](https://codingdog.tistory.com/entry/%ED%8A%B8%EB%A6%AC-dp-%EC%8A%A4%ED%81%AC%EB%A3%A8%EC%A7%80-%EB%AF%BC%ED%98%B8-%EB%AC%B8%EC%A0%9C%EB%A1%9C-%EA%B0%84%EB%8B%A8%ED%95%9C-%EA%B2%83%EB%A7%8C-%ED%92%80%EC%96%B4-%EB%B4%85%EC%8B%9C%EB%8B%A4)
  - [BOJ [1949] 우수 마을](https://private-space.tistory.com/39)
