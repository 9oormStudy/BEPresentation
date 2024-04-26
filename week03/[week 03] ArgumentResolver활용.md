# ArgumentResolver 활용

컨트롤러에서 HTTP Request Body를 변수에 바인딩하기 위해 @RequestBody를 사용하여 ArgumentResolver로 부터 컨트롤러가 필요로 하는 객체를 생성하는 과정을 거친다. ArgumentResolver는 위의 경우 뿐만아니라 HTTP Header, Session, Cookie 등 직접적이지 않은 방식 혹은 외부 데이터 저장소로부터 데이터를 바인딩해야할 때도 사용한다. Argument Resolver를 사용하면 컨트롤러 메서드의 파라미터 중 특정 조건에 맞는 파라미터가 있다면, 요청에 들어온 값을 이용해 원하는 객체를 만들어 바인딩해줄 수 있다. 이번 강의에서 세션에 저장되어 있는 것을 메소드의 파라미터로 넘겨주는 ArgumentResolver를 직접 구현해보겠다.

### 1. ArgumentResolver 패키지 생성

---

### 2. ArgumentResolver에 사용될 Login Annotation 생성

---

```java
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface Login {
}
```

- @Target
  - Annotation을 사용할 수 있는 곳을 가리킨다.
  - Class, Parameter, Method, Field 등등 여러 가지 있다.
- @Retention
  - Annotation의 설정으로 Annotation의 생명주기를 결정한다.
  - Source, Class, Runtime이 있다. 보통 Runtime을 사용하여 프로젝트가 실행 중에도 사용할 수 있도록 Annotation의 생명주기를 정한다.

### 3. ArgumentResolver 생성

---

```java
@Slf4j
public class LoginMemberArgumentResolver implements HandlerMethodArgumentResolver {
	    @Override
	    public boolean supportsParameter(MethodParameter parameter) {
					log.info("supportsParameter 실행");
	        boolean hasLoginAnnotation = parameter.hasParameterAnnotation(Login.class);
	        boolean hasMemberType = Member.class.isAssignableFrom(parameter.getParameterType());
	        return hasLoginAnnotation && hasMemberType;
	    }

			@Override
	    public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer,
			    NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {
					log.info("resolveArgument 실행");
	        HttpServletRequest request = (HttpServletRequest) webRequest.getNativeRequest();
	        HttpSession session = request.getSession(false);
	        if (session == null) {
            return null;
	        }
	        return session.getAttribute(SessionConst.LOGIN_MEMBER);
	    }
}
```

- supportsParameter
  - 우리가 받고자 파라미터가 조건을 만족하는지 확인한다.
    1. @Login Annotation이 붙었는지 확인한다.
    2. 정보를 넣고자 하는 member 클래스인지 확인한다.
- resolveArgument
  - supportsParameter에서 true가 넘어올 시 실행된다.
    1. NativeWebRequest를 HttpServletRequest로 변환한다.
    2. request.getSession(false)로 하여 session이 없다면 생성하지 않는다.
    3. session이 있다면 로그인을 기존에 한 회원이므로 session.getAttribute(SessionConst.LOGIN_MEMBER)를 실행하여 로그인 회원 정보인 member 객체를 찾아 반환해준다.

### 4. MVC에서 사용할 수 있도록 WebConfig에 등록

---

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
		 @Override
     public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
         resolvers.add(new LoginMemberArgumentResolver());
     }
}
```

- 앞서 개발한 LoginMemberArgumentResolver를 등록한다.

### 5. HomeController 변경

---

```java
@GetMapping("/")
public String homeLoginV3ArgumentResolver(@Login Member loginMember, Model model) {
			//세션에 회원 데이터가 없으면 home
			if (loginMember == null) {
	        return "home";
	    }
			//세션이 유지되면 로그인으로 이동
			model.addAttribute("member", loginMember);
			return "loginHome";
}
```

- @Login이 있으면 직접 만든 ArgumentResolver가 동작해서 세션에 있는 로그인 회원을 찾아 회원 정보를 반환하고, 만약 세션에 없다면 null을 반환한다.

### 6. 테스트 해보기

---

홈 화면에 접속 했을 때, 아직 로그인 하기 전이고 세션에 데이터가 없으면
<img width="511" alt="스크린샷 2024-04-15 오후 11 11 28" src="https://github.com/9oormStudy/BEPresentation/assets/159230525/e12f2fb9-ad7c-420b-b6bc-fe26018d5e22">

로그인 후 홈 화면에 접속 했을 때, 세션을 생성해서 반환하면
<img width="600" alt="스크린샷 2024-04-15 오후 11 06 47" src="https://github.com/9oormStudy/BEPresentation/assets/159230525/01eefd08-bd5f-43e7-9d83-b1196b67e616">

<img width="861" alt="스크린샷 2024-04-15 오후 11 11 58" src="https://github.com/9oormStudy/BEPresentation/assets/159230525/a933c009-39d2-4848-aa40-29be62ea24de">
