---
date: 2021-01-18
title: "HashMap과 LinkedHashMap 차이"
categories: Java
---

## 1.1. Map 클래스 다이어그램
Map은 자바에서 제일 많이 쓰이는 자료구조이다. 클래스 다이어그램을 아래와 같이 살펴보자.   
LinkedHashMap은 HashMap을 상속하고 HashMap은 AbstractMap을 상속한다. 그리고 AbstractMap 클래스는 Map 인터페이스를 구현하고 있다.
대강의 클래스 계층도는 아래와 같다. 

<p align="center">
  <img src="/assets/images/2022-01-18-hashmap-linkedhashmap-01.png" height ="30%" width="60%" alt="map diagram"/>
</p>


## 1.2. HashMap과 LinkedHashMap 차이
그러면 언제 HashMap과 LinekdHashMap을 쓰는게 좋을까? 그리고 둘의 성능 차이는 얼마나 날까?
결론적으로 입력받은 데이터에 대해 순서를 유지하고 싶으면 LinkedHashMap을 사용하고 순서가 중요하지 않은 경우 그냥 HashMap을 사용하면 된다.
아래의 코드를 테스트 했을 경우 결과를 쉽게 확인할 수 있다.

```java
Map<String,String> hashMap = new HashMap<>();
hashMap.put("1","aaa");
hashMap.put("2","bbb");
hashMap.put("3","ccc");
hashMap.put("-3","ddd");
hashMap.put("-2","eee");

printMap(hashMap);

Map<String,String> linkedHashMap = new LinkedHashMap<>();
hashMap.put("1","aaa");
hashMap.put("2","bbb");
hashMap.put("3","ccc");
hashMap.put("-3","ddd");
hashMap.put("-2","eee");

printMap(hashMap);
```

HashMap을 사용한 경우 입력한 순서가 유지되지 않는 반면 LinkedHashMap의 경우 입력 순서가 유지되는 것을 확인할 수 있다.
```
//HashMap
1 : aaa
2 : bbb
3 : ccc
-2 : eee  //순서바뀜
-3 : ddd
//LinkedHashMap
1 : aaa
2 : bbb
3 : ccc
-3 : ddd
-2 : eee
```

