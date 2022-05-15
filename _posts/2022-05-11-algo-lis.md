---
layout: post
title: 최장 증가 부분 수열(LIS)
subtitle: Longest Increasing Subsequence
categories: algorithm
tags: [algorithm]
---

- 개인 공부 목적으로 작성한 포스팅입니다.
- 아래 출처를 참고하여 작성하였습니다. :)

### 대표 예시 문제

LIS 유형 문제입니다.

- [반도체 설계](https://www.acmicpc.net/problem/2352)
- [가장 긴 증가하는 부분 수열 5](https://www.acmicpc.net/problem/14003)
- [전깃줄 - 2](https://www.acmicpc.net/problem/2568)

---

### 최장 증가 부분 수열(LIS)이란

원소가 n개인 배열의 일부 원소를 골라내서 만든 부분 수열 중, 각 원소가 이전 원소보다 크다는 조건을 만족하고, 그 길이가 최대인 부분 수열을 최장 증가 부분 수열이라고 합니다.

---

### O(NlogN) 알고리즘

#### lower bound

- 정렬된 배열 arr에서 어떠한 값 `val의 lower bound`란, `arr을 정렬된 상태로 유지`하면서 `val이 삽입될 수 있는 위치들 중 가장 인덱스가 작은 것`을 의미합니다.
- 가령 `[1, 3, 3, 6, 7]`이라는 배열에서 1의 lower bound는 0이고, 3의 lower bound는 1이며, 5의 lower bound는 3입니다.
- lower bound는 이분 탐색을 통해 `O(logN)`에 구할 수 있습니다.

#### 이분탐색을 활용한 LIS

- 시간복잡도를 개선하기 위하여 LIS를 구성할 때 이분탐색을 활용합니다.
- 즉, LIS의 형태를 유지하기 위해 주어진 배열의 인덱스를 하나씩 살펴보면서 그 숫자가 들어갈 위치를 이분탐색으로 찾아서 넣습니다.
- 이를 위해 `lis`라는 리스트를 추가로 사용합니다.

---

- 이미지 출처: [알고리즘 - 최장 증가 부분 수열(LIS) 알고리즘](https://chanhuiseok.github.io/posts/algo-49/)

![lis_refer](https://user-images.githubusercontent.com/75410527/167801236-7dce3ede-6941-4d16-8117-4baf9a02e121.png)

lis 배열 사용 방식

- 주어진 배열을 앞에서부터 순회하면서 다음과 같은 과정을 통해 `lis`를 업데이트합니다.
- `lis의 길이`가 곧 `현재까지 만들 수 있는 LIS의 길이`이며, 처음에는 빈 리스트로 시작합니다.

현재 주어진 배열에서 탐색하는 원소를 `arr[i]`라 표기하겠습니다.

1. lis가 `비어있거나`, `arr[i]`가 lis의 `마지막 원소보다 큰 경우`
   - `lis`의 뒤에 `arr[i]`를 추가합니다.
   - 왜냐면 지금까지 만들 수 있는 가장 긴 증가하는 부분수열의 마지막 원소보다 `arr[i]`가 크기 때문에 `arr[i]`를 마지막 원소로 만들 수 있는 LIS의 길이는 `lis.size()` + 1가 됩니다.
1. 그렇지 않을 경우
   - `lis`에서 `arr[i]의 lowerbound` 위치를 찾고, `arr[i] < lis[lowerbound]`인 경우, `lis[lowerbound]`를 `arr[i]`로 바꿔줍니다.
     - `lis[lowerbound] = arr[i]`

위의 과정으로만 `lis`를 변형하면 정렬된 상태를 깨지 않습니다. 따라서 2번 과정에서 `lowerbound`를 탐색할 때 이분 탐색을 사용할 수 있습니다.
위의 과정을 마치고나서, `lis`의 길이가 이 수열에서 LIS의 길이입니다.

---

### 올바른 LIS 구하기 (+ `int[] indexs`)

- 이분 탐색을 활용하는 단순 LIS 알고리즘으로는 올바른 LIS를 구하지 못할 수도 있습니다.
- 이 알고리즘은 오직 올바른 `LIS 길이 값만 반환`합니다.

그러므로 올바른 LIS를 구하기 위해서는 한 가지 값을 더 저장해야 합니다.

```java
// int[] numbers (주어진 숫자)가 lis에서 어디에 위치하는지 인덱스값을 저장
int[] indexs;
```

1. 주어진 배열을 순회하면서 lowerbound를 구할 때 `indexs[i]` 값도 같이 구해줍니다.

```java
for (int i = 0; i < N; ++i) {
    if (lis.size() == 0) {
        indexs[i] = lis.size();
        lis.add(numbers[i]);
    } else {

        int idx = getLowerBound(numbers[i]);
        int lastIdx = lis.size() - 1;

        if ((idx >= lastIdx) && numbers[i] > lis.get(lastIdx)) {
            indexs[i] = lis.size();
            lis.add(numbers[i]);
        } else {
            if (numbers[i] < lis.get(idx)) {
                lis.set(idx, numbers[i]);
                indexs[i] = idx;
            }
        }
    }
}
```

---

#### lis 길이를 활용해서 역방향 순회

그 다음이 중요합니다.

- 현재 저희는 lis의 길이를 알고 있고, indexs 배열을 통해 각 숫자들이 lis에서 어디에 위치하는지 알 수 있습니다.
- 그러므로 `indexs 배열을 뒤에서부터 순회`하면서 lis에서 맨 뒤에 위치할 수 있는 애부터 하나씩 `answer` 배열에 넣어줍니다.
  - 인덱스에 매칭되는 숫자를 구할 때 마다 찾으려는 인덱스 숫자를 1씩 감소시켜줍니다.
- 그렇게 완성된 `answer` 배열을 뒤집으면 올바른 숫자를 가진 LIS가 완성됩니다.

```java
int targetIdx = lis.size() - 1;

List<Integer> answer = new ArrayList<>();

for (int i = N-1; i >= 0; --i) {
    if (indexs[i] == targetIdx) {
        answer.add(numbers[i]);
        targetIdx--;
    }
}

Collections.reverse(answer);
```

---

### [반도체 설계](https://www.acmicpc.net/problem/2352) 정답 코드

```java
import java.io.*;
import java.util.*;


public class Main {

    public static int N;
    public static List<Integer> L = new ArrayList<>();

    public static void main(String[] args) throws IOException {

        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));

        N = Integer.parseInt(br.readLine());

        String[] connectionInfo = br.readLine().split(" ");

        int[] arr = new int[N];

        for (int i = 0; i < N; ++i) {
            int num = Integer.parseInt(connectionInfo[i]);
            arr[i] = num;
        }

        for (int i = 0; i < N; ++i) {
            if (L.size() == 0) {
                L.add(arr[i]);
            } else {
                // intLow
                int idx = getLowerBound(arr[i]);
                int lastIdx = L.size() - 1;

                if ((idx >= lastIdx) && arr[i] > L.get(lastIdx)) {
                    L.add(arr[i]);
                } else {
                    if (L.get(idx) > arr[i]) {
                        L.set(idx, arr[i]);
                    }
                }
            }
        }

        System.out.println(L.size());
    }

    public static int getLowerBound(int num) {
        int st = 0;
        int en = L.size() - 1;

        int ans = Integer.MAX_VALUE;

        while (st <= en) {
            int mid = (st + en) / 2;
            int midNumber = L.get(mid);

            if (num <= midNumber) {
                ans = Math.min(ans, mid);
                en = mid - 1;
            } else {
                st = mid + 1;
            }
        }

        return ans;
    }
}
```

---

### [가장 긴 증가하는 부분 수열 5](https://www.acmicpc.net/problem/14003) 정답 코드

```java
import java.io.*;
import java.util.*;


public class Main {

    public static int N;
    public static int[] indexs;
    public static List<Integer> lis = new ArrayList<>();

    public static void main(String[] args) throws IOException {

        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));

        N = Integer.parseInt(br.readLine());

        String[] numberInfo = br.readLine().split(" ");

        int[] numbers = new int[N];
        indexs = new int[N];

        for (int i = 0; i < N; ++i) {
            int num = Integer.parseInt(numberInfo[i]);
            numbers[i] = num;
        }

        for (int i = 0; i < N; ++i) {
            if (lis.size() == 0) {
                indexs[i] = lis.size();
                lis.add(numbers[i]);
            } else {

                int idx = getLowerBound(numbers[i]);
                int lastIdx = lis.size() - 1;

                if ((idx >= lastIdx) && numbers[i] > lis.get(lastIdx)) {
                    indexs[i] = lis.size();
                    lis.add(numbers[i]);
                } else {
                    if (numbers[i] < lis.get(idx)) {
                        lis.set(idx, numbers[i]);
                        indexs[i] = idx;
                    }
                }
            }
        }

        int targetIdx = lis.size() - 1;

        List<Integer> answer = new ArrayList<>();

        for (int i = N-1; i >= 0; --i) {
            if (indexs[i] == targetIdx) {
                answer.add(numbers[i]);
                targetIdx--;
            }
        }

        Collections.reverse(answer);

        StringBuilder sb = new StringBuilder();
        sb.append(lis.size()).append("\n");

        for (int number : answer) {
            sb.append(number).append(" ");
        }

        System.out.println(sb);
    }

    public static int getLowerBound(int number) {
        int st = 0;
        int en = lis.size() - 1;
        int ans = Integer.MAX_VALUE;

        while (st <= en) {
            int mid = (st + en) / 2;
            int nowNumber = lis.get(mid);

            if (number <= nowNumber) {
                ans = Math.min(ans, mid);
                en = mid - 1;
            } else {
                st = mid + 1;
            }
        }

        return ans;
    }
}
```

---

### [전깃줄 - 2](https://www.acmicpc.net/problem/2568) 정답 코드

```java
import java.io.*;
import java.util.*;


public class Main {

    public static final int MAX = 500001;
    public static int N;
    public static List<Integer> L = new ArrayList<>();
    public static List<Integer> idxs = new ArrayList<>();
    public static Map<Integer, Integer> valueMap = new HashMap<>();
    public static int[] indexs;

    public static void main(String[] args) throws IOException {

        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));

        N = Integer.parseInt(br.readLine());
        indexs = new int[MAX];
        Arrays.fill(indexs, -1);

        for (int i = 0; i < N; ++i) {
            String[] info = br.readLine().split(" ");
            int idx = Integer.parseInt(info[0]);
            int connection = Integer.parseInt(info[1]);

            idxs.add(idx);
            valueMap.put(idx, connection);
        }

        Collections.sort(idxs);

        for (int idx : idxs) {
            if (L.size() == 0) {
                indexs[idx] = L.size();
                L.add(valueMap.get(idx));
            } else {
                int nowNumber = valueMap.get(idx);
                int lowerBound = getLowerBound(nowNumber);
                int lastIdx = L.size() - 1;

                if (lowerBound >= lastIdx && nowNumber > L.get(lastIdx)) {
                    indexs[idx] = L.size();
                    L.add(nowNumber);
                } else {
                    if (nowNumber < L.get(lastIdx)) {
                        L.set(lowerBound, nowNumber);
                        indexs[idx] = lowerBound;
                    }
                }
            }
        }

        //정답에 포함되는 전깃줄 번호의 리스트
        Set<Integer> answer = new HashSet<>();

        int targetIdx = L.size() - 1;

        for (int i = MAX - 1; i >= 0; --i) {
            if (indexs[i] == targetIdx) {
                answer.add(i);
                targetIdx--;
            }
        }

        List<Integer> jjury = new ArrayList<>();

        for (int i = 0; i < MAX; ++i) {
            if (indexs[i] != -1 && !answer.contains(i)) {
                jjury.add(i);
            }
        }

        StringBuilder sb = new StringBuilder();
        sb.append(jjury.size()).append("\n");

        for (int jjuryNumber : jjury) {
            sb.append(jjuryNumber).append("\n");
        }

        System.out.println(sb.toString());
    }

    public static int getLowerBound(int num) {
        int st = 0;
        int en = L.size() - 1;
        int ans = Integer.MAX_VALUE;

        while (st <= en) {
            int mid = (st + en) / 2;
            int nowNumber = L.get(mid);

            if (num <= nowNumber) {
                ans = Math.min(ans, mid);
                en = mid - 1;
            } else {
                st = mid + 1;
            }
        }

        return ans;
    }
}
```

---

- 참고
  - [알고리즘 - 최장 증가 부분 수열(LIS) 알고리즘](https://chanhuiseok.github.io/posts/algo-49/)
  - [가장 긴 증가하는 부분 수열 (Longest Increasing Subsequence)](https://seungkwan.tistory.com/8)
  - [[ 백준 12738 ] 가장 긴 증가하는 부분수열3 (C++)](https://yabmoons.tistory.com/560)
  - [[ 백준 14003 ] 가장 긴 증가하는 부분수열5 (C++)](https://yabmoons.tistory.com/561)
