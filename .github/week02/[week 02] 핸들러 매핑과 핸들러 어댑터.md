## 핸들러 매핑과 핸들러 어댑터
핸들러 매핑과 핸들러 어댑터가 어떤 것들이 어떻게 사용되는지 알아보기에 앞서 SpringMVC의 구조를 보겠다.

![img](https://images.velog.io/images/hyun6ik/post/d415bfe2-7b57-4ff9-a99b-4292b0abee9b/image.png)

**동작 순서**

**1.** `핸들러 조회:` 핸들러 매핑을 통해 요청 URL에 매핑된 핸들러(컨트롤러)를 조회한다.

**2.** `핸들러 어댑터 조회:` 핸들러를 실행할 수 있는 핸들러 어댑터를 조회한다.

**3.** `핸들러 어댑터 실행:` 핸들러 어댑터를 실행한다.

**4.** `핸`들러 실행:` 핸들러 어댑터가 실제 핸들러를 실행한다.

**5.** `ModelAndView 반환:` 핸들러 어댑터는 핸들러가 반환하는 정보를 ModelAndView로 변환해서 반환한다.

**6.** `viewResolver 호출:` 뷰 리졸버를 찾고 실행한다.

JSP의 경우: InternalResourceViewResolver 가 자동 등록되고, 사용된다.

**7.** `View 반환:` 뷰 리졸버는 뷰의 논리 이름을 물리 이름으로 바꾸고, 렌더링 역할을 담당하는 뷰 객체를 반환한다.

JSP의 경우 InternalResourceView(JstlView) 를 반환하는데, 내부에 forward() 로직이 있다.

**8.** `뷰 렌더링:` 뷰를 통해서 뷰를 렌더링 한다.

### Controller 인터페이스
**과거 버전 스프링 컨트롤러**

 **핸들러 매핑과 어댑터를 이해하기 위해 과거에 주로 사용했던 스프링이 제공하는 간단한 컨트롤러를 살펴보자** (지금은 전혀 사용하지 않음)

**org.springframework.web.servlet.mvc.Controller**
```java
public interface Controller {
ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse
response) throws Exception;
}
```
스프링도 처음에는 이러한 딱딱한 형식의 컨트롤러를 제공함

> 참고 : Controller 인터페이스는 @Controller 애노테이션과는 전혀 다르다



#### OldController
```java
package hello.servlet.web.springmvc.old;
import org.springframework.stereotype.Component;
import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.mvc.Controller;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
@Component("/springmvc/old-controller")
public class OldController implements Controller {
 @Override
 public ModelAndView handleRequest(HttpServletRequest request,
HttpServletResponse response) throws Exception {
 System.out.println("OldController.handleRequest");
 return null;
 }
}
```
- `@Component: ` 이 컨트롤러는 `/springmvc/old-controller`라는 이름의 스프링 빈으로 등록됨
- 빈의 이름으로 URL을 매핑할 것이다
- 실행해보면 콘솔에 `OldController.handleRequest`이 출력됨

#### 스프링 mvc 구조

![img](https://images.velog.io/images/hyun6ik/post/d415bfe2-7b57-4ff9-a99b-4292b0abee9b/image.png)


이 컨트롤러가 호출되려면 다음 2가지가 필요하다

**1. HandlerMapping(핸들러 매핑)**

- 핸들러 매핑에서 이 컨트롤러를 찾을 수 있어야 한다.
- ex: 스프링 빈의 이름으로 핸들러를 찾을 수 있는 핸들러 매핑이 필요하다.

**2. HandlerAdapter(핸들러 어댑터)**

- 핸들러 매핑을 통해서 찾은 핸들러를 실행할 수 있는 핸들러 어댑터가 필요하다.
- ex: Controller 인터페이스를 실행할 수 있는 핸들러 어댑터를 찾고 실행해야 한다.

> **사실 스프링은 이미 필요한 핸들러 매핑과 핸들러 어댑터를 대부분 구현해두었다. 개발자가 직접 핸들러 매핑과 핸들러 어댑터를 만드는 일은 거의 없다.**


### 스프링 부트가 자동 등록하는 핸들러 매핑과 핸들러 어댑터

### HandlerMapping

0 = `RequestMappingHandlerMapping :` 애노테이션 기반의 컨트롤러인 @RequestMapping에서 사용

1 = `BeanNameUrlHandlerMapping :` 스프링 빈의 이름으로 핸들러를 찾는다

### HandlerAdapter

0 = `RequestMappingHandlerAdapter :` 애노테이션 기반의 컨트롤러인 @RequestMapping에서 사용

1 = `HttpRequestHandlerAdapter :` HttpRequestHandler 처리

2 = `SimpleControllerHandlerAdapter :` Controller 인터페이스(애노테이션X, 과거에 사용) 처리

**핸들러 매핑도, 핸들러 어댑터도 모두 순서대로 찾고 만약 없으면 다음 순서로 넘어간다**

### 1. 핸들러 매핑으로 핸들러 조회
---
1. `HandlerMapping` 을 순서대로 실행해서, 핸들러를 찾는다.

2. 이 경우 빈 이름으로 핸들러를 찾아야 하기 때문에 이름 그대로 빈 이름으로 핸들러를 찾아주는 `BeanNameUrlHandlerMapping` 가 실행에 성공하고 핸들러인 `OldController` 를 반환한다.

### 2. 핸들러 어댑터 조회
---
1. `HandlerAdapter` 의 `supports()` 를 순서대로 호출한다.
2. `SimpleControllerHandlerAdapter` 가 Controller 인터페이스를 지원하므로 대상이 된다.

### 3. 핸들러 어댑터 실행
---

1. 디스패처 서블릿이 조회한 `SimpleControllerHandlerAdapter` 를 실행하면서 핸들러 정보도 함께 넘겨준다.
2. `SimpleControllerHandlerAdapter` 는 핸들러인 `OldController` 를 내부에서 실행하고, 그 결과를 반환한다

### 정리 - OldController 핸들러매핑, 어댑터
---
OldController를 실행하면서 사용된 객체는 다음과 같다.

`HandlerMapping = BeanNameUrlHandlerMapping`

`HandlerAdapter = SimpleControllerHandlerAdapter`

### HttpRequestHandler

- 핸들러 매핑과, 어댑터를 더 잘 이해하기 위해 Controller 인터페이스가 아닌 다른 핸들러를 알아보자.

- `HttpRequestHandler 핸들러(컨트롤러)`는 서블릿과 가장 유사한 형태의 핸들러이다.

### HttpRequestHandler

```java
public interface HttpRequestHandler {
void handleRequest(HttpServletRequest request, HttpServletResponse response)
throws ServletException, IOException;
}
```

### MyHttpRequestHandler // 구현한 코드
```java
package hello.servlet.web.springmvc.old;
import org.springframework.stereotype.Component;
import org.springframework.web.HttpRequestHandler;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
@Component("/springmvc/request-handler")
public class MyHttpRequestHandler implements HttpRequestHandler {
 @Override
 public void handleRequest(HttpServletRequest request, HttpServletResponse
response) throws ServletException, IOException {
 System.out.println("MyHttpRequestHandler.handleRequest");
 }
}
```

- 실행해보면 웹 브라우저에 빈 화면이 나오고, 콘솔에 `MyHttpRequestHandler.handleRequest`가 출력된다

### 1. 핸들러 매핑으로 핸들러 조회
1. `HandlerMapping` 을 순서대로 실행해서, 핸들러를 찾는다.

2. 이 경우 빈 이름으로 핸들러를 찾아야 하기 때문에 이름 그대로 빈 이름으로 핸들러를 찾아주는 `BeanNameUrlHandlerMapping`가 실행에 성공하고 핸들러인 `MyHttpRequestHandler`를 반환한다.

### 2. 핸들러 어댑터 조회
1. `HandlerAdapte`r의 supports()를 순서대로 호출한다.

2. `HttpRequestHandlerAdapter`가 `HttpRequestHandler `인터페이스를 지원하므로 대상이 된다.

### 3. 핸들러 어댑터 실행
**1. 디스패처 서블릿이 조회한 `HttpRequestHandlerAdapter`를 실행하면서 핸들러 정보도 함께 넘겨준다.**

**2. `HttpRequestHandlerAdapter`는 핸들러인 `MyHttpRequestHandler`를 내부에서 실행하고, 그 결과를 반환한다**

#### 정리 - MyHttpRequestHandler 핸들러매핑, 어댑터

**`MyHttpRequestHandler` 를 실행하면서 사용된 객체는 다음과 같다.**

`HandlerMapping = BeanNameUrlHandlerMapping`
`HandlerAdapter = HttpRequestHandlerAdapter`

### RequestMapping
- 가장 우선순위가 높은 핸들러 매핑과 핸들러 어댑터는 `RequestMappingHandlerMapping`, `RequestMappingHandlerAdapter` 이다.

- `@RequestMapping`의 앞글자를 따서 만든 이름인데, 이것이 바로 지금 스프링에서 주로 사용하는 애노테이션 기반의 컨트롤러를 지원하는 매핑과 어댑터이다. 

- 실무에서는 99.9% 이 방식의 컨트롤러를 사용한다고 한다.


