---
date: 2021-01-31
title: "@ComponentScan 이해하기"
categories: Spring
---


## 1.1. @ComponentScan 동작순서

* @ComponentScan은 @Component가 붙은 모든 클래스를 빈으로 등록한다.
  * 클래스의 이름의 첫 글자를 소문자로 하여 빈이름으로 등록하고 클래스의 인스턴스를 빈 객체로 등록.
* 생성자에 @Autowired를 지정하면 스프링 컨테이너가 자동으로 빈 저장소에서 객체랑 같은 타입을 가진 빈을 등록해 준다.
* 탐색할 빈의 위치를 지정해 줄 수 있다.
  * 아래와 같이 설정하면 hello.core 패키지 밑에 @Component가 붙은 모든 빈을 스프링 빈으로 등록한다.
```java
@ComponentScan(
          basePackages = "hello.core",
}
```
* 

***** 
## References
