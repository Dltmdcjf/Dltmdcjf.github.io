---
date: 2021-01-29
title: "SortedSet과 TreeSet"
categories: java
---

## 1. 개요
* SortedSet과 TreeSet에 대해서 알아보자

## 2.1 Set 계층구조
* 먼저 SortedSet은 인터페이스이며 TreeSet은 클래스이다.
* 아래의 계층 구조를 확인하자.
<p align="center">
  <img src="/assets/images/2022-01-29-sortedset-treeset-01.png" height ="100%" width="100%" alt="Set hierarchy"/>
</p>
<p align="center">
    [Set 다이어그램]
</p>
* TreeSet -> NaviagableSet -> SortedSet -> Set 순서로 인터페이스를 구현하고 있다.
  
## 2.2. SortedSet
* Set은 중복을 제거한 원소들을 저장할 때 사용한다.
* SortedSet은 원소들이 정렬되어 있는 Set이다.
  * 정렬을 하는 기준이 필요한데, 이를 위해 `Comparable` 또는 `Comparator` 인터페이스를 사용해야 한다.
  * `Comparable` 인터페이스를 구현하는 클래스의 객체를 원소로 사용하거나
  * `Comparator` 인터페이스를 구현한 대소판단 로직을 SortedSet 객체 생성 시에 넣어주면 된다.
* SortedSet은 내부적으로 Comparator 인터페이스를 가지고 있다. 
  > Integer, String 의 클래스는 `Comparable` 인터페이스를 구현하고 있기 때문에 별도로 정의하지 않아도 알아서 기본 정렬을 한다.

* 아래 코드는 주어진 string에 대해서 3개씩 자른 다음 알파벳 순서로 가장 먼저 나오는 문자열과 가장 나중에 나오는 문자열을 출력하고 있다.
  * SortedSet을 사용하면 자동으로 알파벳 오름차순으로 정렬하는 것을 확인할 수 있다. (String은 내부적으로 comparable을 구현하고 있기 때문에)
```java
        String s = "teststringabc";
        int k = 3;
        int len = s.length() - k + 1;

        SortedSet<String> sorted = new TreeSet<>();

        for (int i = 0; i < len; i++) {
            String substring = s.substring(i, i + k);
            sorted.add(substring);
        }
        System.out.println(sorted.first());
        System.out.println(sorted.last());
```

## 2.3. TreeSet

* TreeSet은 SortedSet에 NavigableSet 인터페이스를 상속한 클래스이다.  (SortedSet의 구현체라고 보면된다.)
  * 따라서 역할은 동일하며 더 많은 메소드를 제공한다.
* TreeSet은 정렬과 관련된 메소드를 제공한다.
* 아래 코드는 TreeSet에 삽입된 원소를 내림차순 혹은 오름 차순으로 정렬하는 것을 보여준다.
* `descendingSet()`을 통해 NavigableSet 인터페이스를 활용하는 것을 볼 수 있다.
```java
        TreeSet<Integer> treeSet = new TreeSet<>();
        treeSet.add(100);
        treeSet.add(34);
        treeSet.add(234);
        treeSet.add(1040);
        treeSet.add(43);
        treeSet.add(2);

        System.out.println("TreeSetMain.main desc");
        NavigableSet<Integer> descSet = treeSet.descendingSet();
        for(Integer descending : descSet) {
            System.out.println(descending);
        }

        System.out.println("TreeSetMain.main asc");
        NavigableSet<Integer> ascSet = descSet.descendingSet();
        for(Integer asc : ascSet) {
            System.out.println(asc);
        }
```


***** 
## References
* [자바 SortedSet](<https://heedipro.tistory.com/241>)