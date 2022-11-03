---
layout: post
title: Collections.emptyList() vs new ArrayList<>()
subtitle: immutable vs mutable
categories: todayilearned
tags: [todayilearned]
---

- 개인 공부 목적으로 작성한 포스팅입니다.
- 아래 출처를 참고하여 작성하였습니다. :)

### Collections.emptyList()

> 수정 불가능한 빈 리스트입니다.

- `명시적으로 '추가 삽입이 없는 그냥 빈 리스트'를 만들고 싶을 때` Collections.emptyList()는 그 의도를 더 잘 표현했습니다.

그래서 아래와 같이 `Collections.emptyList()`를 호출해서 `add` 연산자를 실행해보면 예외가 발생합니다.

```java
public static void main(String[] args) throws IOException {
    List<Integer> immutableIntegerList = Collections.emptyList();

    for (int loop = 0; loop < 5; ++loop) {
        immutableIntegerList.add(loop); // 예외 발생
    }
}
```

이렇게 `UnsupportedOperationException` 예외가 발생합니다.

```
Exception in thread "main" java.lang.UnsupportedOperationException
	at java.util.AbstractList.add(AbstractList.java:148)
	at java.util.AbstractList.add(AbstractList.java:108)
	at Solution.main(Solution.java:13)

Process finished with exit code 1
```

- `AbstractList` 추상 클래스는 `random access`를 구현하는데 필요한 최소한의 것만 만들어 `List` 인터페이스의 구현체를 제공합니다.

> 그래서 add 메서드 호출 시 `UnsupportedOperationException` 예외가 발생함을 알 수 있습니다.

```java
//AbstractList.java

//caller
public boolean add(E e) {
    add(size(), e);
    return true;
}

//callee
public void add(int index, E element) {
    throw new UnsupportedOperationException();
}
```

#### 빈 리스트 딱 하나만 생성

공식문서를 보면 아래와 같은 문장이 있습니다.

> Collection.emptyList() creates a new empty list instance only once

그리고 아래와 같이 `emptyList()` 메서드를 사용해서 `EMPTY_LIST 인스턴스를 재사용`합니다.

```java
public static final List EMPTY_LIST = new EmptyList<>();

public static final <T> List<T> emptyList() {
    return (List<T>) EMPTY_LIST;
}
```

그러므로 `명시적으로 빈 목록을 만들고 싶을 때`는 `Collection.emptyList()`를 사용하는 것이 바람직합니다.

---

- 출처
  - [Collections.emptyList() vs. New List Instance](https://www.baeldung.com/java-collections-emptylist-new-list)
