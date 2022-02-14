---
date: 2021-01-19
title: "Spring의 @Configuration과 싱글톤"
categories: Spring
---

## 1.1. @Configuration 애노테이션
스프링의 @Configuration 애노테이션은 어떤 역할을 하는지 알아보자.
만약 아래와 같은 클래스가 있다고 하자. 두 개의 메서드에서는 각각 AService 와 BService 객체를 생성한다. 
그리고 내부적으로 createRepository를 호출하는데 createRepository안에서는 TestRepository 객체를 생성한 후 리턴하고 있다.

```java
@Configuration
public class AppConfig {
    @Bean
    AService createServiceA() {
        return new AService(createRepository());
    }

    @Bean
    BService createServiceB() {
        return new BService(createRepository());
    }

    @Bean
    TestRepository createRepository() {
        return new TestRepository();
    }
}
```

스프링 컨테이너는 싱글톤으로 동작한다고 알고 있다. 그럼 과연 위의 코드에 있는 3개의 메소드를 한 번씩 수행하면 `new TestRepository()`를 총 3번 수행 하게 되는데,
과연 만들어진 `TestRepository`는 같은 주소값을 가지고 있는지 확인해 보자. ApplicationContext를 이용해 AppConfig를 스프링 컨테이너로 등록하자.   
3개의 Bean이 메소드의 이름으로 만들어 졌을테고 AService,BService 그리고 createRepository로 만들어진 TestRepository 객체가 같은 값을 같는지 확인 해보자

```
    @Test
    void singleBeanTest() {
        ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
        TestRepository testRepository = ac.getBean(TestRepository.class);
        AService aService = ac.getBean(AService.class);
        BService bService = ac.getBean(BService.class);

        System.out.println(testRepository);
        System.out.println(aService.getTestRepository());
        System.out.println(bService.getTestRepository());
    }
```

코드를 수행하고 나서 나온 결과값이다. 놀랍게도 같은 값을 가지는 것을 확인할 수 있다. 어떻게 이것이 가능할까? 그것은 바로 @Configuration 애노테이션에 있다.
```
spring.configuration.TestRepository@3b2c0e88
spring.configuration.TestRepository@3b2c0e88
spring.configuration.TestRepository@3b2c0e88
```

클래스 위에 @Configuration 애노테이션을 설정하면 스프링이 해당 클래스의 바이트 코드를 조작한다. 즉 해당 클래스의 바이트 코드를 조작하는데 만약 동일한 객체가
이미 스프링 컨테이너에 존재하면, 또 객체를 생성하지 않고 있는 객체를 그대로 리턴한다. 이런 방식으로 모든 객체는 단 한 개의 인스턴스를 유지할 수 있다. 


## 2. 정리
@Bean만 있으면 스프링 Bean으로 등록하지만 싱글톤은 보장하지 않는다. 따라서 스프링 설정정보는 무조건 @Configuration을 붙어서 싱글톤을 보장하도록 하자.