---
layout: post
title: 알고리즘별 풀이 쇼츠
subtitle: 본인이 볼려고 정리해놓는 풀이 쇼츠
categories: ps
tags: [ps]
---

- 필요한대로 계속 업데이트할 예정입니다 :)

### LIS

최장 증가 부분 수열입니다.

#### LIS 길이 구하기

- 아래 세 가지 조합으로 최장 증가 부분 수열의 `길이`를 구할 수 있습니다.
  - `lower bound` + `이분 탐색` + `List<> lis`
- `lower bound`는 `현재 숫자가` lis 리스트에서 `위치할 수 있는 가장 작은 인덱스를 의미`합니다.
- 이분 탐색은 위 lower bound를 O(logN) 시간에 구하기 위해 사용합니다.
- 주어진 배열 내 숫자를 하나씩 탐색하면서 이분 탐색으로 `lower bound`를 구하며 `List<> lis`를 갱신해주면 됩니다.
  - 맨 뒤에 넣을 수 있으면 맨 끝에 add
  - 중간에 넣을 수 있으면 기존의 `lis[index]` 값 갱신

예시 코드

```java
static int N;
static int[] numbers;
static List<Integer> lis = new ArrayList<>();

public static void main(String[] args) {

    for (int num : numbers) {
        int index = getLowerBound(num);

        if (index == lis.size()) {
            lis.add(num);
        } else {
            int comparedValue = lis.get(index);

            if (num < comparedValue) {
                lis.set(index, num);
            }
        }
    }

    System.out.println(lis.size());
}

static int getLowerBound(int value) {
    int st = 0;
    int en = lis.size() - 1;
    int ans = lis.size();

    while (st <= en) {
        int mid = (st + en) / 2;
        int comparedValue = lis.get(mid);

        if (value <= comparedValue) {
            ans = mid;
            en = mid - 1;
        } else {
            st = mid + 1;
        }
    }

    return ans;
}
```

### DP

#### 동전 고르기 & 냅색

- 동전 고르기
  - N개의 동전 `int[] coins`를 가지고 M원을 만들 수 있는 동전의 최대/최소 갯수
  - 핵심은 N,M을 어떤 순서로 루프 도는지가 중요하고, 새로운 `i번째 동전`을 사용할 때마다 `0~M원`을 전부 확인해줘야합니다.

```java
/**
    * dp[i]: N번째 동전까지 고려했을 때 M원을 만드는데 필요한 최대/최소 동전 갯수
*/
int[] coins;

for (int i = 0; i < N; ++i) {
    for (int m = 0; m <= M; ++m) {
        if (m >= coins[i]) {
            dp[m] = Math.max(dp[m], dp[m-coins[i]] + 1);
            //OR
            dp[m] = Math.min(dp[m], dp[m-coins[i]] + 1);
        }
    }
}
```

- 냅색 문제
  - 핵심은 `dp[i][m]과의 비교대상`은 가방에 `넣거나, 안 넣거나` 이렇게 두 가지 상태입니다.
    - 가방에 안 넣거나: `dp[i][m]`
    - 가방에 넣거나: `dp[i][m-weight] + value`
  - 자기 자신의 최대값을 갱신하는 dp[i][m] = Math.max(dp[i][m], newValue) 이런 형태가 아닙니다.

```java
/**
    * dp[i][j]: i번째 보석까지 고려하고, 무게 j만큼 담았을 때 얻을 수 있는 최대 가치
    *
    * dp[i][j] = dp[i-1][j-nowWeight] + nowValue
    * dp[i][j] = d[i-1][j]
    */
for (int i = 1; i <= N; ++i) {
    for (int m = 0; m <= M; ++m) {
        Stone nowStone = stones[i];

        if (m >= nowStone.weight) {
            dp[i][m] = Math.max(dp[i-1][m], dp[i-1][m-nowStone.weight] + nowStone.value);
        } else {
            dp[i][m] = dp[i-1][m];
        }

        ans = Math.max(ans, dp[i][m]);
    }
}
```

---

- 참고
  - [[알고리즘 트레이닝] 5장 - 동적계획법과 냅색(Knapsack) (백준 12865번 평범한 배낭 문제로 살펴보기)](https://chanhuiseok.github.io/posts/improve-6/)
