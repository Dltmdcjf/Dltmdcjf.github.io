---
date: 2021-01-16
title: "JDK, JRE 그리고 JIT 컴파일러"
categories: Java 
---

# 1. JDK, JRE, JVM, JIT 정의
## 1.1. JRE
* 자바 에플리케이션을 실행할 수 있도록 구성된 배포판
* JRE에는 javac가 존재하지 않는다.
* JRE는 Runtime 환경에서 자바를 실행할 수 있지만 javac가 존재하지 않기 때문에 .java를 컴파일하여 .class 파일을 만들수는 없다.

## 1.2. JDK
* JRE + 개발에 필요한 툴
* 오라클은 자바 11부터 JDK만 제공하며 JRE를 따로 제공하지 않는다.

## 1.3 JVM
* 자바 가상 머신으로 자바 바이크 코드를 인터프리터와 JIT 컴파일러를 사용하여 OS에 특화된 코드로 변환한다. 

### 1.3.1 JVM 구조
![JVM_STRUCTURE](/assets/images/2022-01-16-jdk-jre-jvm-jit_02.png){: width="70%" height="70%"}{: .align-center}
* 클래스 로더
  1. 로더 : 자바 바이트 코드를 읽어 들인 후 메모리에 적절하게 배치하는 역할
  2. 링커 : 링크를 사용해 레퍼런스를 연결하는 과정
  3. 초기화 : static 값들의 초기화 및 변수의 할당
  > 클래스 로더는 기본적으로 계층형태를 가지는 3가지 로더가 제공된다. 부트스트랩, 플랫폼, 애플리케이션 클래스 로더. 
  맨 처음 제일 부모인 부트스트랩 로더가 클래스를 로드한다. 그리고 플랫폼, 애플리케이션 순으로 클래스를 로드한다.
  3가지 로더에서 클래스를 로드하지 못할 경우 ClassNotFoundException 에러가 난다.
  * 아래 예제는 DeckTest 클래스의 클래스로더를 확인하는 예제이다. 실행결과 DeckTest의 클래스로더는 App 클래스로더이고
  App 클래스 로더의 부모 클래스 로더는 플랫폼 클래스 로더인것을 확인할 수 있다.
  ```java
  public class DeckTest {
    public static void main(String[] args) {
        ClassLoader classLoader = DeckTest.class.getClassLoader();
        System.out.println(classLoader);
        System.out.println(classLoader.getParent());
    }
  }
  ```
  * 실행결과
  ```
  jdk.internal.loader.ClassLoaders$AppClassLoader@2c13da15
  jdk.internal.loader.ClassLoaders$PlatformClassLoader@7d417077
  jdk.internal.loader.ClassLoaders$PlatformClassLoader@7d417077
  ```
* JVM 메모리
  * 메소드 영역 : static 변수, class 및 부모 클래스 정보등이 저장되는 공유자원
  * 힙 영역 : 인스턴스(객체)들을 저장
  * 스택 : 쓰레드 마다 런타임 스택을 만들고, 그 안에 메소드 호출 스택을 쌓는다. 
  * PC 레지스터 : 쓰레드 마다 쓰레드 내 현지 실행할 스택 프레임을 가리키는 포인터를 갖고있다.
  * 네이티브 메소드 스택 : 네이티브 메소드를 사용한 경우에 유지되는 스택.
* 실행엔진
  * 인터프린터 : 바이트 코드를 읽고 네이티브 코드로 보내주는 역할
  * JIT 컴파일러 : 인터프리터의 효율을 높이기 위해 반복적인 바이트 코드는 모두 네이티브 코드로 바꾸어 준다. 인터프린터가 반복적인 바이트 코드를 만났을 때,   
  기존에 미리 바꾸어 둔 네이티브 코드로 사용한다.
  * GC : 참조되지 않는 객체를 모아서 정리하는 역할.

## 1.4 JIT 컴파일러
* Java는 예전에 인터프리터하는 시간 때문에 속도가 느리다고 공격을 항상 받아왔다.
  * 더욱이 인터프리터는 런타임 동안 한줄 한줄 읽어 들어야 하는 방식이기 때문에 느린건 만연한 사실이다.
* 그래서 속도 지연을 방지하기 위해 Java 프로그램에서 JIT 컴파일러는 JAVA 바이트 코드를 프로세서에 직접 전송할 수 있는 명령어로 전환하는 역할을 한다.
* 쉽게 이야기하면 JIT 컴파일러는 기존에 번역한 바이트 코드에 대해서는 번역하지 않고 그대로 저장소에 있는 내용을 가져와 기계어로 번역한다.

![JIT](/assets/images/2022-01-16-jdk-jre-jvm-jit_01.png){: width="70%" height="70%"}{: .align-center}

***** 

## References
* [JVM 구조](<https://www.guru99.com/java-virtual-machine-jvm.html>)
* [JIT 컴파일러](<https://aboullaite.me/understanding-jit-compiler-just-in-time-compiler/>)