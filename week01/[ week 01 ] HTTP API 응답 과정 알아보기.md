# HTTP API 응답 과정 알아보기

```java
package com.example.demo.controller;

@RestController
@Slf4j
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        log.info("Hello Controller call");
        return "hello";
    }
}

```
@RestController 를 사용하는 컨트롤러에 접근하면 어떤 흐름으로 응답이 생성되는지  디버깅을 통해 알아보자.

---

#### DispatcherServlet - doDispatch
![](https://velog.velcdn.com/images/gltdhd/post/aee8e766-0fca-48cf-a368-fd604c6542af/image.png)

모든 요청은 FrontController 인 DispatcherServlet 이 담당하므로 클래스의 doDispatch 메서드에 브레이크 포인트를 걸고 디버그 모드로 실행하고, http://localhost:8080/hello 를 요청하면 실행 흐름을 알 수 있다.


---

#### DispatcherServlet - getHandle 핸들러 조회
![](https://velog.velcdn.com/images/gltdhd/post/053c5263-2035-420c-a909-788a2af602fe/image.png)

getHandler() 메서드를 통해 핸들러 조회한다.


![](https://velog.velcdn.com/images/gltdhd/post/dce2393d-25c2-4d62-b958-d3074bbf18db/image.png)

메서드 내부 코드를 보면 HandlerMappings 변수에 기본적으로 위와 같은 핸들러 매핑 정보들이 구현되어 있는 것을 확인할 수 있다.


![](https://velog.velcdn.com/images/gltdhd/post/d9d49635-b021-408e-b028-3982b2558e52/image.png)

어노테이션 기반의 컨트롤러를 사용했으므로 RequestMappingHandlerMapping 이 HandlerMapping 으로 선택된다. 이 핸들러 매핑으로 컨트롤러 객체가 핸들러로 리턴되는 것을 확인할 수 있다.


---

#### DispatcherServlet - getHandlerAdapter 핸들러 어댑터 조회
![](https://velog.velcdn.com/images/gltdhd/post/3cded9f0-adc6-430f-85b1-83338e29ce8e/image.png)

핸들러를 찾은 후 getHandlerAdapter 를 통해 해당 핸들러를 처리할 수 있는 핸들러 어댑터를 찾는 과정으로 넘어간다.

![](https://velog.velcdn.com/images/gltdhd/post/cbb3d42e-0c0e-4b24-b963-b8fb3f4f01a9/image.png)

핸들러 매핑을 조회하는 과정과 비슷하게 스프링의 기본적인 핸들러 어댑터가 구현되어 있는 것을 확인할 수 있고, 어노테이션 기반의 컨트롤러이므로 RequestMappingHandlerAdapter 가 선택되어 반환된다.

----

#### RequestMappingHandlerAdapter - 핸들러 어댑터 실행
![](https://velog.velcdn.com/images/gltdhd/post/4ff0a93a-4db7-4e26-bcb5-133ddb9535d9/image.png)

이제 찾아낸 핸들러 어댑터 (RequestMappingHandlerAdapter) 를 통해 핸들러를 실행하는 메소드 handle() 가 작동한다.

더 자세히 보기위해 handle() 메서드 내부로 들어가면 handleInternal() 메서드를 거쳐 invokeHandlerMethod() 메서드로 이어지게 되는데, 전달받은 컨트롤러의 메서드를 여기서 실행하게 된다.

![](https://velog.velcdn.com/images/gltdhd/post/adb17835-e053-4ab4-853c-ecbb06b4a165/image.png)

이 메서드는  컨트롤러 메서드에 전달된 요청 파라미터, 경로 변수, 요청 본문 등의 데이터를 추출하고, 메서드 매개변수에 바인딩할 수 있는 객체로 변환하는 작업을 담당하는 **ArgumentResolver** 와

컨트롤러 메서드의 반환 유형에 따라 다르게 작동하며, 반환된 객체를 HTTP 응답으로 변환하거나, ModelAndView 객체를 생성하는 **HandlerMethodReturnValueHandler** 가 작동하는 곳이다.

( 더 자세한 작동 로직 디버그 과정은 너무 길어서 생략...)

- **ArgumentResolver** : 어떠한 요청 파라미터도 없으므로 선택되지 않음
- **ReturnValueHandler** : @ResponseBody 어노테이션을 처리할 수 있는
RequestResponseBodyMethodProcessor 가 선택되고 HTTP 메시지 컨버터를 통해 HTTP 응답 메세지 본문에 데이터를 생성


#### DispatcherServlet
![](https://velog.velcdn.com/images/gltdhd/post/8f0f8ab8-57c5-47ef-b091-569c0282bc38/image.png)

DispatcherServlet 는 최종적으로 ModelAndView 객체를 반환받지만, view가 필요하지 않으므로 비어있는 상태로 전달받는다. 

클라이언트는 이미 메세지 컨버터를 통해 HTTP 응답 메세지를 받았으므로 비어있는 ModelAndView 객체를 응답하지 않도록 구현되어 있다.



# 정리

일반적인 뷰 템플릿을 사용하는 경우 다음과 같이 디스패처 서블릿이 ModelAndView 객체를 반환받아서, viewResolver 를 호출하고 HTML 응답을 하는 과정이 필요하다.

![](https://velog.velcdn.com/images/gltdhd/post/bd037715-3a1b-4e70-b8cd-63f8a693d4d0/image.png)

하지만 HTTP API 응답의 핵심 과정은 다음과 같이RequestResponseBodyMethodProcessor 와 HttpMessageConverter 를 통해 직접 응답 본문을 생성하고 전송하므로 View 를 렌더링 하는 과정이 생략된다.
 

![](https://velog.velcdn.com/images/gltdhd/post/f221c216-f4a0-451e-9147-a9c980c94781/image.png)