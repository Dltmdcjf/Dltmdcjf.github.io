---
date: 2021-01-20
title: "Spring MVC 패턴 학습"
categories: Spring
---

## 1. MVC 패턴의 발전

### 1.1. 서블릿과 자바 코드만으로 구현
 
 자바 코드와 서블릿만으로 HTML 페이지를 구성할 수 있다. 서블릿으로 request 요청을 받은 후
 서블릿에서 비지니스 로직을 실행한 후 (회원가입, 조회등의 기능) 직접 HTML 페이지를 자바코드로 작성하여 response를 통해
 HTML 화면을 제공할 수 있다.   
다음과 같이 HttpServlet을 상속받은 클래스는 서블릿 코드를 작성할 수 있다.   
`/servlet/test/`에 request 요청이 들어오면 `service` 메소드가 수행이 되는데, 아래 코드에서 getWriter()를 통해
직접 HTML 코드를 작성하고 있다. 이렇게 해당 메서드가 종료되는 시점이면 작성한 HTML 코드가 요청 주소에 렌더링되는 것을 확인할 수 있다.
이렇게 직접 서블릿과 자바 코드만으로 구현할 수 있으며 서블릿 덕분에 동적으로 화면에 렌더링 할 수 있다.
하지만 코드에서 보듯이 매우 복잡하고 비효율적이다. 한 메서드에서 비지니스 로직과 화면을 렌더링 그리고 요청과 응답을 받는 부분까지 모두 처리하고 있는 것을
확인할 수 있다.

 ```java
@WebServlet(name = "userListServlet", urlPatterns = "/servlet/test")
public class UserListServlet extends HttpServlet {
    private MemberRepository repository = Repository.getInstance();

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html");
        response.setCharacterEncoding("utf-8");
        List<User> users = repository.findAll();
        PrintWriter w = response.getWriter();
        w.write("<html>");
        w.write("<head>");
        w.write("    <meta charset=\"UTF-8\">");
        w.write("    <title>Title</title>");
        w.write("</head>");
        w.write("<body>");
        w.write("<a href=\"/index.html\">메인</a>");
        ...

        for (User user : users) {
            w.write("    <tr>");
            w.write("     <td> " + user.getId() + "</td>");
            w.write("     <td> " + user.getUsername() + "</td>");
            w.write("     <td> " + user.getAge() + "</td>");
            w.write("   </tr>");
        }
        w.write("    </tbody>");
        w.write("</table>");
        w.write("</body>");
        w.write("</html>");
    }
}
```

### 1.2. JSP 사용
1.1에 자바와 서블릿을 사용하는 것 보다 JSP를 사용하면 화면 담당하는 부분을 좀 더 분리할 수 있다.
직접 JSP를 작성한 후 여기에 HTML을 이용해 화면을 구성하고 데이터를 동적으로 보여주는 부분에 대해서 JSP 문법을 사용해
HTML 중간에 자바 코드를 작성할 수 있다. 이렇게 되면 보다 코드를 분리할 수 있게 된다.


#### 1.2.1. JSP 한계
JSP도 결국에는 HTML 과 JAVA 코드가 같이 섞여 있게 된다. 한 쪽에는 비지니스 로직이 있는 자바 코드와 다른 한쪽에는 화면을 담당하는
HTML 코드가 뒤섞이게 된다. 이런 경우 비지로직이 .jsp 파일에 노출이 되게 되고 코드의 양이 많아지면 질수록 유지보수가 어려워진다.
그리고 어찌됐든 자바코드와 HTML 코드가 같이 썩여있는것이 보기에 안좋다.

```jsp
<%
    Repository repository = Repository.getInstance();
    List<User> users = repository.findAll();
%>
<html>
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
..
```

### 1.3. MVC 패턴 사용

MVC 패턴의 가장 필요한 부분은 유지보수가 안되는 점도 있지만, JSP 처럼 관리하면 두 영역의 변경 싸이클이 다르다는 점이다.
UI를 수정하는 일과 비지니스 로직의 싸이클은 서로 독립적이고 변경 싸이클도 다른데 두 영역을 한 곳에서 모아 처리하는 것은 유지보수를 힘들게 만든다.
이런 관점에서 MVC 패턴이 등장하였다.

#### 1.3.1 MVC 패턴이란

* Model : 뷰에 출력할 데이터를 담는 곳
* View : 모델에 담겨 있는 데이터를 사용해 화면을 출력하는 곳
* Controller : HTTP 요청을 받은 후 비지니스 로직을 실행하는 곳

<p align="center">
  <img src="/assets/images/2022-01-20-spring-mvc-study-01.png" height ="30%" width="60%" alt="mvc pattern"/>
</p>
<p align="center">
    <a href="https://ko.wikipedia.org/wiki/%EB%AA%A8%EB%8D%B8-%EB%B7%B0-%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC#/media/%ED%8C%8C%EC%9D%BC:MVC-Process.svg">[출처] MVC pattern</a>
</p>

#### 1.3.2. MVC 패턴 한계
MVC 패턴도 한계가 있는데 대표적으로 컨트롤러에 중복되는 코드가 많다는 점이다. 예를 들어 View로 이동하는 코드는 모든 컨트롤러마다 중복되어 구성된다.
그리고 response를 사용하지 않는 경우도 많으며, 공통된 처리에 대한 부분이 미약하다. 
```
RequestDispatcher dispatcher = request.getRequestDispatcher(view);
dispatcher.forward(request, response);
```

### 1.4. Front Controller 도입
이를 해결하기 위해 컨트롤러마다 공통된 부분을 Front Controller라는 녀석이 해주면 중복된 부분을 없앨 수 있다. 그렇게 함으로써 공통된 처리가 가능해진다.


# 정리
단순히 서블릿과 Java만으로 처리하다가 JSP 도입 그리고 MVC 패턴까지 개념적으로 발전해왔다. 그리고 단순히 MVC 패턴을 그냥 이용하는게
아니라 Front Controller의 도입으로 공통 부분을 Front Controller에서 처리하니 중복되는 코드를 줄이는게 가능해졌다.
이런 부분이 발전되어 현재의 Spring MVC 구조가 되게 된다.