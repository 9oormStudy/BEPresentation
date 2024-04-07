# HTTP 메시지 컨버터

---

HTTP 요청에 의해 메시지 바디를 직접 읽거나 응답에 의해 메시지 바디를 직접 작성할 때 사용하는 컨버터이다.

뷰 템플릿으로 HTML을 생성해서 응답하는 것이 아니라, JSON 데이터를 HTTP 메시지 바디에서 직접 읽거나 쓰는 경우 **HTTP 메시지 컨버터**를 사용하면 편리하다.

### 메시지 컨버터 적용 대상

---

- HTTP 요청: @RequestBody, HttpEntity(RequestEntity)
- HTTP 응답: @ResponseBody, HttpEntity(ResponseEntity)

### HttpMessageConverter 인터페이스

---

<img width="602" alt="스크린샷 2024-04-07 오후 10 09 01" src="https://github.com/9oormStudy/BEPresentation/assets/159230525/06a76f68-7b9e-426b-9ce2-4d4c124df9b2">

HTTP 메시지 컨버터는 HTTP 요청, HTTP 응답 둘 다 사용된다.

- canRead(), canWrite()
  - 메시지 컨버터가 해당 클래스와 미디어 타입을 지원하는지 확인
- read(), write()
  - 메시지 컨버터를 이용해 메시지를 읽고 쓰는 기능

### **메세지 컨버터의 종류**

---

- 요청 시

```java
/0 = ByteArrayHttpMessageConverter // 클래스 타입 : byte[] , 미디어 타입 : */*
//요청 예시) @RequestBody byte[] data

1 = StringHttpMessageConverter // 클래스 타입 : String , 미디어 타입 : */*
//요청 예시) @ReuqestBody String data

2 = MappingJackson2HttpMessageConverter // 클래스 타입 : 객체 또는 HashMap , 미디어 타입 : application/json
//요청 예시) @RequestBody Data(직접 만든 객체) data
```

- 응답 시

```java
0 = ByteArrayHttpMessageConverter // 미디어 타입 : application/octet-stream
//응답 예시) @ResponseBody return byte[]

1 = StringHttpMessageConverter // 미디어 타입 : text/plain
//응답 예시) @ResponseBody return "ok";

2 = MappingJackson2HttpMessageConverter // 미디어 타입 : application/json
//응답 예시) @ResponseBody return data
```

### 메시지 컨버터 동작

---

```java
@ResponseBody
@RequestMapping(value="/hello", method=RequestMethod.POST)
public Stringhello(@RequestBody String param){
	return "result";
}
```

- HTTP 요청 데이터 조회
  1. HTTP 요청이 왔을 때 컨트롤러에서 @RequestBody 파라미터를 사용
  2. 메시지 컨버터가 메시지를 읽을 수 있는지 확인하기 위해 canRead()를 호출
     (이때 canRead의 파라미터로 대상 클래스와 Content-Type 미디어 타입을 전달)
  3. canRead() 조건을 만족하면 read()를 호출해서 객체를 생성하고 반환
  4. @RequestBody 파라미터에 바인딩
- HTTP 응답 데이터 생성
  1. 컨트롤러에서 @ResponseBody 값이 반환
  2. 메시지 컨버터는 자신이 메시지를 쓸 수 있는지 확인하기 위해 canWrite()를 호출
     (이때 canWrite의 파라미터로 대상 클래스와 HTTP 요청 헤더의 Accept 미디어 타입을 파라미터로 전달)
  3. canWrite() 조건을 만족하면 write()를 호출하여 HTTP 응답 메시지 바디에 데이터를 생성

### **HTTP 메시지 컨버터는 스프링 MVC 어디 지점에서 사용될까?**

---

@RequestMapping 을 처리하는 핸들러 어댑터인 RequestMappingHandlerAdapter 에서 사용된다.

# **RequestMappingHandlerAdapter**

---

<img width="553" alt="스크린샷 2024-04-08 오전 12 38 37" src="https://github.com/9oormStudy/BEPresentation/assets/159230525/dcc21dbb-20ca-4fe7-a4fe-41eab1de2d0c">
<img width="553" alt="스크린샷 2024-04-08 오전 12 14 47" src="https://github.com/9oormStudy/BEPresentation/assets/159230525/70f13340-6b09-407d-a4ff-745167b87ea4">

RequestMappingHandlerAdapter는 @RequestMapping 어노테이션으로 선언된 Handler(Controller)를 지원하는 Adapter이다.

### HandlerAdapter 동작 과정

![스크린샷 2024-04-08 오전 12 58 22](https://github.com/9oormStudy/BEPresentation/assets/159230525/2d7f85b1-ef30-46e1-9c7d-970c5c1e0af8)

### Argument Resolver란?

---

핸들러 어댑터로부터 애노테이션 기반 컨트롤러를 처리하는 **ArgumentResolver**(=HandlerMethodArgumentResolver)를 호출해서 컨트롤러(=핸들러)가 필요로 하는 객체를 생성하고 처리

```java
   public interface HandlerMethodArgumentResolver {
      boolean supportsParameter(MethodParameter parameter);

  	@Nullable
      Object resolveArgument(MethodParameter parameter, @Nullable
   ModelAndViewContainer mavContainer,
               NativeWebRequest webRequest, @Nullable WebDataBinderFactory
   binderFactory) throws Exception;
  }
```

(-> supportsParameter()를 호출해서 해당 파라미터를 지원하는지 확인하고, 지원하면 resolveArgument()를 호출해서 실제 객체를 생성하고 컨트롤러 호출 시 객체를 넘긴다)

### ReturnValueHandler란?

---

핸들러에서 애노테이션 기반 컨트롤러를 처리하는 **ReturnValueHandler** 를 호출해서 핸들러 어댑터가 필요한 응답 값을 변환하고 처리

# 전체 동작 과정

---

<img width="547" alt="스크린샷 2024-04-07 오후 11 14 35" src="https://github.com/9oormStudy/BEPresentation/assets/159230525/f25dbbdc-b84c-4b81-8bdb-f4d81c22a8eb">

- 요청 처리 시
  1. HandlerMapping을 통해 적절한 HandlerAdapter를 찾으면 HandlerAdapter는 Controller로 넘겨줄 파라미터를 결정하기 위해 이 작업을 ArgumentResolver에게 위임
  2. ArgumentResolver는 HTTP Request Body를 HttpMessageConverter에게 특정 타입의 객체로 변환을 요청하여 특정 타입의 객체로 변환
  3. ArgumentResolver는 변환된 데이터를 전달 받아서 이 데이터를 다시 HandlerAdapter에게 전달
  4. HandlerAdapter는 전달 받은 데이터를 핸들러 메서드의 파라미터로 포함 시킨 후, 핸들러 메서드를 호출
- 응답 처리 시
  1. 핸들러 메서드가 응답으로 전달할 데이터를 리턴
  2. ReturnValueHandler는 핸들러 메서드로부터 전달 받은 응답 데이터를 HttpMessageConverter에게 전달하여 HTTP Response Body에 포함되는 형식의 데이터로 변환
  3. ReturnValueHandler는 HttpMessageConverter로부터 전달 받은 데이터를 HandlerAdapter에게 전달

### 출처

---

https://lordofkangs.tistory.com/487

[https://maenco.tistory.com/entry/Spring-MVC-HTTP-Message-Converters-HTTP-메시지-컨버터](https://maenco.tistory.com/entry/Spring-MVC-HTTP-Message-Converters-HTTP-%EB%A9%94%EC%8B%9C%EC%A7%80-%EC%BB%A8%EB%B2%84%ED%84%B0)

인프런 - 김영한 스프링 MVC
