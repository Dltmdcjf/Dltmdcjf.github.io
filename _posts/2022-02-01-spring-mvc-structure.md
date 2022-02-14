---
date: 2021-02-01
title: "스프링 MVC 구조"
categories: Spring
---


## 1.1. Spring MVC 구조

### 2.1 DispatcherServlet 이란?
* DispatcherServlet은 스프링의 핵심이며 프론트 컨트롤러의 역할을 한다.
  * DispatcherServlet -> FrameworkServlet -> HttpServletBean -> HttpServlet
* 스프링 부트는 내장 톰캣에서 DispatcherServlet을 서블릿으로 자동으로 등록하면 스프링 부트를 띄운다. 그리고 모든 경로 ("/") 에 대해서
전부 DispatcherServlet이 호출된다.
  * `FrameworkServlet`에서 service()를 오버라이드 하였다. 이 service()가 처음으로 호출되고 DispatcherServlet.doDispatch()가 호출되는데
  이 doDispatch()가 Spring MVC 구조에서 핵심이다.
* 동작순서는 아래와 같다.
  * 핸들러 조회 : 핸들러 매핑에서 핸들러(컨트롤러)를 조회한다.
  * 핸들러 어댑터 조회 : 핸들러를 실행할 수 있는 핸들러 어댑터를 조회한다.
  * 핸들러 어댑터 실행 : 핸들러 어댑터에서 핸들러를 실행한다.
  * 핸들러 실행 : 핸들러 어댑터가 핸들러를 실행.
  * ModelAndView 반환 : 핸들러 어댑터는 핸들러를 실행해서 ModelAndView를 반환한다.
  * ViewResolver 호출 : ViewResolver를 호출해서 View를 반환한다.
  * 뷰 렌더링

<p align="center">
  <img src="/assets/images/2022-02-01-spring-mvc-structure-01.png" height ="100%" width="100%" alt="spring mvc structure"/>
</p>
<p align="center">
    <a href="https://programmer.help/blogs/this-is-the-introduction-to-spring-mvc.html">[출처] 스프링 MVC 구조</a>
</p>


### 2.2. HandlerMapping과 HandlerAdapters 호출 순서
* HandlerMapping (컨트롤러를 찾는 순서)
  * 핸들러매핑은 @RequestMapping 애노테이션이 있는 컨트롤러를 먼저 찾는다.
  * 없을 경우에 BeanName으로 핸들러를 찾는다.
    * 0 = RequestMappingHandlerMapping : 애노테이션 기반의 컨트롤러 @RequestMapping
    * 1 = BeanNameUrlHandlerMapping : 스프링 빈의 이름으로 핸들러를 찾는다.
* HandlerAdapters (어댑터를 찾아 핸들러를 호출하는 순서)
  * 핸들러 어댑터도 @ReqeustMapping 먼저 찾은 후 1,2번 순서로 어댑터를 찾고 호출한다.
    * 0 = RequestMappingHandlerAdapter : 애노테이션 기반의 컨트롤러 @RequestMapping
    * 1 = HttpRequestHandlerAdapter : HttpRequestHandler 처리
    * 2 = SimpleControllerHandlerAdapter : Controller 인터페이스(애노테이션X) 처리

* 핸들러매핑과 핸들러 어댑터가 호출되는 순서
  * HandlerMapping 순서에 의해 컨트롤러를 찾는다.
  * 핸들러를 실행할 수 있는 어댑터를 핸들러 어댑터 순서에 의해 찾는다.
  * 핸들러 어댑터 순서에 의해 어댑터를 찾게되면 handle() 메소드를 수행하여 핸들러를 실행한다. 
  * 이때 handle() 메소드를 수행하는 곳이 바로 DispatcherServlet 이다.


### 2.3. @Controller와 @RequestMapping 이해하기

```java
@Controller
  public class SpringMemberFormControllerV1 {
      @RequestMapping("/springmvc/v1/members/new-form")
      public ModelAndView process() {
          return new ModelAndView("new-form");
      }
          }
```

* 위의 코드가 어떻게 동작하는지 이해해보자
  * @Controller 애노테이션에 의해 스프링이 자동으로 스프링 빈으로 등록한다.
  * RequestMappingHandlerMapping에서 @Controller 혹은 @RequestMapping이 있으면 매핑 정보(애노테이션 기반 컨트롤러)로 인식한다.
  * @RequestMapping에 의해 해당 URL을 이 메서드로 매핑하여 호출한다.

* 요약
  * HTTP 요청이 온다.
  * Dispatcher Servlet에서 핸들러 매핑에서 핸들러를 조회해서 순차적으로 찾아서 처리할 수 있는 핸들러를 찾아준다.
  * 핸들러 어댑터 목록에 던져주면 해당 핸들러를 처리할 수 있는 핸들러 어댑터가 튀어 나온다. 
  * 그래서 핸들러 어댑터를 통해서 실제 핸들러를 호출한다.
  * 반환은 모델엔뷰를 반환하고 뷰리졸버를 호출해서 실제 뷰를 찾는다.
  * HTTP 고객 응답에게 나가게 된다.



***** 
## References
