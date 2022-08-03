---
layout: post
title: 알고리즘별 풀이 쇼츠
subtitle: 본인이 볼려고 정리해놓는 풀이 쇼츠
categories: ps
tags: [ps]
---

- 필요한대로 계속 업데이트할 예정입니다 :)

### 이진탐색

#### Lower bound

- 원하는 `target 이상의 값`이 `최초로 나오는 위치`를 뜻합니다.
- 달리 말하면, `target보다 같거나 큰 원소`의 위치 중 가장 작은 위치를 의미합니다.

```java
int getLowerBound(int[] numbers, int val) {
    int st = 0;
    int en = N-1;
    int lowerBound = IMPOSSIBLE;

    while (st <= en) {
        int mid = (st + en) / 2;

        if (numbers[mid] >= val) {
            lowerBound = Math.min(lowerBound, mid);
            en = mid - 1;
        } else {
            st = mid + 1;
        }
    }

    return ans;
}
```

#### Upper bound

- 원하는 `target을 초과하는 값`이 `최초로 나오는 위치`를 뜻합니다.
- 달리 말하면, `target을 초과하는 원소`의 위치 중 가장 작은 위치를 의미합니다.
- Lower bound 코드에서 `등호만 제외하면` Upper bound를 구하는 코드입니다.

```java
int getUpperBound(int[] numbers, int val) {
    int st = 0;
    int en = N-1;
    int upperBound = IMPOSSIBLE;

    while (st <= en) {
        int mid = (st + en) / 2;

        if (numbers[mid] > val) {
            upperBound = Math.min(upperBound, mid);
            en = mid - 1;
        } else {
            st = mid + 1;
        }
    }

    return ans;
}
```

#### 배열 내 데이터의 갯수

- lower bound와 upper bound의 성질을 이용하여 배열 내 데이터의 갯수를 구할 수 있습니다.
- 배열 내 데이터의 갯수는 `lowerBound`와 `upperBound` 이 두 값이 `차이`입니다.
- `데이터가 존재하지 않는다면` 두 값은 같습니다.

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

#### 공통 유의사항

- dp의 특정상태 `dp[i][j]`가 `몇 번 계산(정의)되는지` 따져봐야 합니다.

`여러 번` 계산이 된다면

- dp 갱신값의 `비교 대상은 자기 자신`이 됩니다.
  - 왜냐면 같은 상태가 여러번 정의될 수 있으므로 그 중에서의 최댓값/최솟값을 구해야 하기 때문입니다.

```java
//자기 자신과 비교연산
dp[i][j] = Math.max(dp[i][j], ...);
```

`단 한 번`만 계산된다면

- dp 갱신값의 `비교 대상은 자신의 이전 상태`가 됩니다.
  - 왜냐면 특정 상태는 `딱 한 번만 계산` 되므로 자신과 비교할 필요가 없고 이전 상태와 현재 상태 중 뭐가 더 나은지만 판단하면 됩니다.

```java
//이전 상태와 비교연산
dp[i][j] = Math.max(dp[i-1][j], ...);
```

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

### 위상정렬

#### 기본 풀이 순서

1. 가장 기본적인 풀이 순서는 그래프를 만들면서 `나에게 들어오는 간선의 갯수`를 세는 `int[] indegree`를 계산합니다.
2. `모든` `indegree[i] == 0인 정점을 큐에 넣고` 탐색을 시작합니다.
3. 큐에서 정점을 하나씩 빼면서 인접한 모든 정점을 탐색하고 그 때 마다 `--indegree[nxt]`을 해주면서 `indegree[nxt] == 0`이 되면 해당 정점을 큐에 넣어줍니다.

[BOJ 2056번: 작업](https://www.acmicpc.net/problem/2056) 문제 풀이 코드

```java
    static int N;
    static int[] indegree, times;
    static List<Integer>[] graph;

    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));

        N = Integer.parseInt(br.readLine());
        indegree = new int[N];
        times = new int[N];

        graph = new List[N];
        for (int i = 0; i < N; ++i) {
            graph[i] = new ArrayList<>();
        }

        for (int i = 0; i < N; ++i) {
            String[] jobInfo = br.readLine().split(" ");
            int time = Integer.parseInt(jobInfo[0]);
            int preCnt = Integer.parseInt(jobInfo[1]);

            times[i] = time;

            for (int j = 0; j < preCnt; ++j) {
                int preJobNumber = Integer.parseInt(jobInfo[j+2]);
                preJobNumber--;
                graph[preJobNumber].add(i);
                //1. 그래프를 만들면서 `나에게 들어오는 간선의 갯수`를 세는 `int[] indegree`를 계산
                indegree[i]++;
            }
        }

        int[] dist = new int[N];
        Queue<Integer> q = new LinkedList<>();

        //2. `모든` `indegree[i] == 0`인 정점을 큐에 넣고 탐색을 시작
        for (int i = 0; i < N; ++i) {
            if (indegree[i] == 0) {
                dist[i] = times[i];
                q.add(i);
            }
        }


        while (!q.isEmpty()) {
            int now = q.poll();

            for (int nxt : graph[now]) {
                if (dist[nxt] < dist[now] + times[nxt]) {
                    dist[nxt] = dist[now] + times[nxt];
                }

                /**
                 * 3. 큐에서 정점을 하나씩 빼면서 인접한 모든 정점을 탐색하고
                 * 그 때 마다 --indegree[nxt]을 해주면서 indegree[nxt] == 0이 되면 해당 정점을 큐에 넣어줍니다.
                 */
                if (--indegree[nxt] == 0) {
                    q.add(nxt);
                }
            }
        }

        int ans = 0;
        for (int i = 0; i < N; ++i) ans = Math.max(ans,dist[i]);

        System.out.println(ans);
    }
}
```

#### 위상 정렬 최대 길이 구하기

- 주어진 그래프를 위상 정렬로 정렬했을 시 `가장 오래 걸리는 경로`를 찾는 유형입니다.
- 다익스트라랑 비슷하면서도 다른 부분이 있어서 정리합니다.
- 아래는 `위상 정렬 내 최대 길이`를 구하는 로직 코드입니다.
- 기존과 달라서 중요한 부분은 `4가지`입니다.
  1. `indegree[i] == 0`인 시작점을 `한 번에 큐에 다 넣습니다.`
  2. `최대 길이`를 구해야 하므로 `dist[i]`는 0 같은 `작은 값으로 초기화`합니다.
  3. `dist[nxt]` 갱신 조건은 아래와 같이 `"최댓값을 바꿀 수 있을 때"` 입니다.
  - `if (dist[nxt] < dist[now] + times[nxt]) {}`
  4. 마지막으로 `큐에 새로 원소가 들어가는 조건` 입니다.
  - 매 루프마다 `indegree[nxt]`값을 감소시키고 `indegree[nxt] == 0`이면 큐에 새로 넣습니다.

```java
int[] dist = new int[N];
Queue<Integer> q = new LinkedList<>();

for (int i = 0; i < N; ++i) {
    if (indegree[i] == 0) {
        dist[i] = times[i];
        q.add(i);
    }
}

while (!q.isEmpty()) {
    int now = q.poll();

    for (int nxt : graph[now]) {
        if (dist[nxt] < dist[now] + times[nxt]) {
            dist[nxt] = dist[now] + times[nxt];
        }

        if (--indegree[nxt] == 0) {
            q.add(nxt);
        }
    }
}
```

#### 경로 칠하기

- 위상 정렬 내 최대 길이에 `포함되는 경로의 갯수`를 구하는 문제입니다.
- 예시 문제: [BOJ 1948번: 임계경로](https://www.acmicpc.net/problem/1948)

풀이 방법은 다음과 같습니다.

1. 위상 정렬로 주어진 그래프 `정방향으로` 순회하면서 `dist[i]` 완성하기
2. 그 다음에 `경로를 칠하는 방법`은 `역방향으로 정점을 순회`하면서 아래 조건을 만족하는 경우만 `dfs로 따라가면 됩니다.`

- `if (dist[node] == dist[nxtNode] + nxtTime) {}`
- 해당 조건은 '이 간선'은 첫 번째 단계에서 구해놓은 `'dist[]가 최댓값으로 이동하는 경로에 포함되는 간선'`이라는 뜻입니다.

3. 단, 주의할 점은 `이미 방문한 정점을 재방문하는 경우`가 생깁니다.

- 그러므로 무조건 `dfs(nxtNode)`를 호출해서 방문할 수 있게 처리하되, 방문했다면 호출 후 바로 종료시키는 방식으로 처리해야 합니다.

```java
void dfs(int node) {
    //중복 방문 제거
    if (visit[node]) return;
    visit[node] = true;

    for (Integer[] nxt : reverseGraph[node]) {
        int nxtNode = nxt[0];
        int nxtTime = nxt[1];
        if (dist[node] == dist[nxtNode] + nxtTime) {
            dfs(nxtNode);
            ans++;
        }
    }
}
```

### DIJKSTRA

#### 특정 간선만 길이를 2배로

특정 간선의 길이만 `2배로 늘렸을 때` 최단 경로가 어떻게 달라지는지 물어보는 문제입니다.

- 풀이의 핵심은, 2배로 늘렸을 때 최단 경로가 달라지게 하는 간선은 `기존의 최단 경로에 포함되는 간선들만 해당된다`는 점입니다.
- 즉, 기존에 최단 경로에 없던 간선은, 길이가 2배로 늘어나도 정답이었던 최단 경로에 아무런 영향을 끼치지 않습니다.
- 풀이방법
  1. 다익스트라를 한 번 돌려서 최단 경로에 포함되는 간선들을 구합니다.
  2. 최단 경로에 포함되는 간선들만 하나씩 뽑아 2배로 늘려보면서 새로 최단 경로를 구해봅니다.
- 여기서 중요한 건, 최단 경로에 포함되는 간선은 `N-1`개 이므로 시간복잡도는 `O(N * MlogM)`이 됩니다.
  - 정점의 갯수: N개, 간선의 갯수: M개라고 가정합니다.
  - 시간복잡도가 `O(M * MlogM)`이 아니라는 것이 중요합니다.

### 우선순위 큐 사용문제

우선순위 큐를 이하 PQ로 표현합니다.

- PQ를 사용해서 `2개 이상의 조건`으로 주어진 데이터를 처리해야 하는 경우
- 이 경우 풀이 방법은 1개의 조건으로 주어진 데이터를 `정렬`합니다.
- 그 후 정렬된 데이터를 `루프를 돌면서`, 매 루프마다 예외 상황에 대해 `PQ`가 처리하도록 합니다.
- 위 풀이는 `O(N^2)`이면 터지는 문제를 `O(NlogN)`으로 바꾸려고 하는 문제입니다.
- 정렬하는데 `O(NlogN)`, 루프 도는데 `O(N)`, 루프마다 PQ를 사용하게 되므로 `O(NlogN)`입니다.
- `최악의 문제 풀이`는 `PQ를 2개 이상 사용`하면서 서로 값을 옮겨담는 것입니다.
  - 이 풀이는 `O(N^2)`입니다.

---

- 참고
  - [[알고리즘 트레이닝] 5장 - 동적계획법과 냅색(Knapsack) (백준 12865번 평범한 배낭 문제로 살펴보기)](https://chanhuiseok.github.io/posts/improve-6/)
  - [BOJ 1948 - 임계경로 (위상정렬)](https://suuntree.tistory.com/120)
  - [BOJ 2056 - 작업 (위상정렬)](https://suuntree.tistory.com/119)
