---
date: 2021-01-16
title: "Enum은 싱글톤 클래스"
categories: Java 
---

# 1. Enum(열거형)에 관하여
## 1.1. Enum의 정의와 이해
```java
enum Card {CLOVER,HEART,SPADE,DIAMOND}
```
Enum이란 서로 관련된 상수를 편리하게 선언하기 위한 것으로 여러 상수를 정의할 때 사용하면 편리하다. 위와 같이 간단하게 `enum` 키워드를 작성한 후 클래스명과 상수들을
나열해서 작성하면 된다. 사실 위 정의는 상수 하나하나가 Card의 객체가 된다. 즉 위의 enum Card는 아래와 같이 다시 고쳐 쓸 수 있다.
     
```java
class Card {
  static final Card CLOVER = new Card("CLOVER");
  static final Card HEART = new Card("HEART");
  static final Card SPADE = new Card("SPADE");
  static final Card DIAMOND = new Card("DIAMOND");

  private String name;

  private Card(String name) {
    this.name = name;
  }
}
```
Enum에서 `==`연산이 가능한데 그 이유는 위와 같이  `new`연산을 사용해서 객체를 생성하지만 `static`을 사용해서 JVM에서 모든 객체가 해당 상수를 공유하기 때문에
단 한 개의 주소값을 가지고 있기 때문에 이 값은 바뀌지 않는 유일한 값이다. 따라서 `==` 연산이 가능한 것이다. 

> Enum은 특별한 형태의 클래스이다. 따라서 생성자를 가져야 하는데, `private` 생성자를 가지는걸 제한한다. 런타임 동작 때 값이 변경되는 것을 막기 위한 것이다.

## 1.2. Enum은 특별한 형태의 클래스이며 싱글톤 성질을 갖는다.
위의 정의에서 살펴보았듯이 Enum은 상수만을 가지며 `private` 생성자로 인스턴스의 생성을 제한한다. 이러한 특성은 싱글톤의 성질과 동일하다.   
Singleton 패턴을 구현한 클래스와 Enum을 사용한 것을 비교해 보자.


1. 싱글톤
```java
public class Singleton{
    private static final Singleton INSTANCE = new Singleton();
    private Singleton(){}
    public static Singleton getSingleton(){
        return INSTANCE;
    }
}
```

2. Enum
```java
enum Singleton {INSTANCE}
```

### 1.2.1. 싱글톤 패턴을 Enum을 사용한 장점
1. 간단하다.
```
Enum의 인스턴스는 thread-safe 하다. 따라서 thread 관련하여 프로그래밍을 할 필요가 없다. 
```
2. Serialization 문제를 스스로 해결한다.
```
싱글톤 패턴의 경우 직렬화 할 때 몇 가지 주의사항이 있다. 직렬화 할 경우 readObject()를    
구현해야 하는데, readObject()는 매번 새로운 인스턴스를 반환하기 때문에 이를 해결하기    
위해 싱글톤 패턴에서는 몇 가지 작업을 추가해야 한다.   
1) implement Serializable 추가    
2) 모든 필드를 transient로 선언    
3) readResolve() 메소드 구현
하지만 위의 모든 과정은 Enum에서 불필요하다.
```

### 1.3 Enum의 static block
아래와 같은 코드가 있다고 할 때 호출되는 순서를 생각해보자.

```java
enum Enums
{
    A, B, C;
    {
       System.out.println(1);
    }
     
    static
    {
        System.out.println(2);
    }
     
    private Enums()
    {
        System.out.println(3);
    }
}
 
public class MainClass
{
    public static void main(String[] args)
    {
        Enum en = Enums.C;
    }
}
```
어떻게 호출되기를 기대하는가? 나는 클래스가 로딩될 때 static 영역부터 메모리 구조에서 메소드 영역에 자리 잡기 때문에 당연히 2가 먼저 출력될 줄 알았다.
하지만 결과는 초기화 블록과 생성자 순으로 출력 후 맨 마지막으로 static영역 안에 있는 내용이 실행된다. 그 이유는 구글리을 좀 해보니
enum의 경우 `{}` 블록이 제일 먼저 수행된다. 왜냐하면 내부적으로 전부 static final 이기 때문이다. 그리고 나서 두 번째로 `static {}` 블록이 실행된다.
음 잘 안와닿지만 기억은 해두자.

***** 

## References
* [Enum 싱글톤](<https://stackoverflow.com/questions/23721115/singleton-pattern-using-enum-version>)
* [Enum 호출순서](<https://stackoverflow.com/questions/11419519/enums-static-and-instance-blocks>)