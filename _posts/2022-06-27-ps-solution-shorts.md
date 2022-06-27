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
- `lower bound`는 현재 숫자가 `lis` 리스트에서 위치할 수 있는 가장 작은 인덱스를 의미합니다.
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
