---
date: 2021-01-20
title: "Arrays.asList의 UnsupportedOperationException 문제"
categories: Java
---

## 1.1. Arrays.asList 사용시 UnsupportedOperationException 예외
코딩하다 보면 스트링을 배열로 바꿔줄 때가 많다. 그럴 때 간편하게 Arrays.asList를 사용하게 되는데
List로 바꿔준 후 만약 remove를 호출하게 되면 `UnsupportedOperationException` 예외가 터진다.
아래 테스트 코드 처럼 배열을 리스트로 바꿔준 후 remove 호출하면 예외가 터졌다.
```java
    @Test
    void listTest() {
        String[] strings = {"key","aaa","Bbb"};

        List<String> strings1 = Arrays.asList(strings);
        strings1.remove(0);
    }
```
```
java.lang.UnsupportedOperationException
	at java.base/java.util.AbstractList.remove(AbstractList.java:167)
	at list.ListTestTest.listTest(ListTestTest.java:26)
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
```

## 1.2. 원인
원인은 Arrays.asList를 사용하면 fixed array를 갖기 때문에 그렇다. 아래 처럼 적힌 것을 내부 코드에서 확인할 수 있다.
> Returns a fixed-size list backed by the specified array.

## 1.3. 해결방법
해결방법은 한번더 ArrayList로 감싸주면 끝난다.

```java
List<String> strings1 = new ArrayList<>(Arrays.asList(strings));
```