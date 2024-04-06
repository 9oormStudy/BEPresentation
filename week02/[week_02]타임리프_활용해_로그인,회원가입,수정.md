## 타임리프를 활용해 회원가입,로그인,수정 만들기

> 타임리프는 주로 서버 측 자바 웹 애플리케이션 개발에 사용되는 템플릿 엔진.

타임리프는 서버측에서만 사용되는 것이 아니고, 이메일 템플릿, pdf생성 xml 처리 등 다양한 곳에서 활용될 수 있다. 하지만 주로 자바 웹 애플리케이션에서 사용되는 것이 흔하며, 이를 위해 많은 서버 측 웹 프레임워크에서 타임리프를 지원한다.

### 타임리프 특징
- 서버 사이드 HTML 렌더링
- 네츄럴 템플릿
- 스프링 통합 지원

### 장점
1. 유연한 문법: Thymeleaf의 문법은 자유롭게 사용할 수 있다. 따라서 기존의 HTML 코드에 추가하기가 쉬우며, 다른 템플릿 엔진과 마찬가지로 자바 코드와 결합하여 사용할 수 있다.
2. 자바 객체와 바인딩: Thymeleaf는 자바 객체와 HTML을 쉽게 연결할 수 있다. 또한 자바 객체의 필드나 메서드를 사용하여 HTML 코드를 생성할 수 있다.
3. Spring Framework와의 연동: Thymeleaf는 Spring Framework와 통합되어 사용할 수 있습니다. Spring MVC와 함께 사용하면 자동화된 폼 처리, 메시지 다국어 처리, 검증 및 페이징과 같은 기능을 쉽게 구현할 수 있다.
4. 간단한 템플릿 캐시: Thymeleaf는 템플릿의 변경을 감지하여 자동으로 캐시를 갱신합니다. 이렇게 하면 템플릿 변경 사항이 즉시 적용되어 애플리케이션의 성능이 향상된다.
5. 다양한 템플릿 모드: Thymeleaf는 HTML을 생성하는 표준 모드 외에도 텍스트 및 XML 모드를 지원합니다. 이러한 모드는 다양한 종류의 템플릿을 지원한다.

### 속성

|th:text:|HTML 태그의 텍스트를 수정합니다.|
|th:if| 지정된 조건이 참인 경우 HTML 요소를 표시합니다.|
|th:unless| 지정된 조건이 거짓인 경우 HTML 요소를 표시합니다.|
|th:switch| 지정된 조건에 따라 여러 HTML 요소 중 하나를 표시합니다.|
|th:case| th:switch와 함께 사용되어 해당 case의 조건이 참인 경우 해당 HTML 요소를 표시합니다.|
|th:each| 반복문을 실행하여 HTML 요소를 생성합니다.|
|th:object| HTML 태그에 객체를 바인딩합니다.|
|th:field| 폼 필드에 데이터를 바인딩합니다.|
|th:action| 폼의 액션 URL을 설정합니다.|
|th:method| 폼의 HTTP 메서드를 설정합니다.|
|th:attr| HTML 태그의 속성을 수정합니다.|
|th:replace| 지정된 템플릿을 현재 요소에 대체합니다.|
|th:remove| HTML 요소를 제거합니다.|
|th:with| 변수를 정의하고 값에 대한 참조를 제공합니다.|


### 도메인 구성

~~~java
	private Long id;

	@NotEmpty(message = "사용자 이름을 작성해주세요 !")
	private String username;

	@NotEmpty(message = "사용할 아이디를 작성해주세요 !")
	private String loginId;

	@NotEmpty(message = "비밀번호를 입력해주세요 !")
	private String password;

	@Enumerated(EnumType.STRING)
	@NotNull(message = "성별을 선택해 주세요 !")
	private Gender gender;

	@Pattern(regexp = "###-####-####", message = "전화번호는 000-0000-0000으로 입력해주세요")
	private String phoneNumber;

	private String email;

	private String address;

~~~

### 타임리프로 레이아웃 지정
**Header.html**
~~~html
<div>
    <br>
    <a th:href="@{/}">
        <div class="header_logo">
            <h1>알바마켓</h1>
        </div>
    </a>
    <div class="header_nav">
        <a th:href="@{/boards/board_list}"><span>커뮤니티</span></a> |
        <a th:if="${isLogin}" th:href="@{/timetable/save_schedule}"><span>시간표관리 |</span></a>
        <a th:if="${isLogin}" th:href="@{/members/edit}"><span>정보수정 |</span></a>
        <a th:if="${isLogin}" th:href="@{/logout}"><span>로그아웃</span></a>
        <a th:unless="${isLogin}" th:href="@{/login}"><span>로그인 |</span></a>
        <a th:unless="${isLogin}" th:href="@{/members/add}"><span>회원가입</span></a>
    </div>
</div>

~~~
**Head.html**
~~~html
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" th:href="@{https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css}"
          integrity="sha384-ggOyR0iXCbMQv3Xipma34MD+dH/1fQ784/j6cY/iJTQUOhcWr7x9JvoRxT2MZw1T" crossorigin="anonymous">
    <link rel="preconnect" th:href="@{https://fonts.gstatic.com}" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Fuzzy+Bubbles&family=Poor+Story&family=Single+Day&display=swap" rel="stylesheet">
    <link rel="stylesheet" th:href="@{/css/main.css}">
    <link rel="stylesheet" th:href="@{/css/test.css}">

    <title>알바 마켓</title>
</head>

~~~
**index.html**
~~~html
<html lang="ko" xmlns:th="http://www.thymeleaf.org"
                xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity5">
<head th:replace="~{common/head :: head}"></head>
<link rel="stylesheet" th:href="@{/css/pagemain.css}">
<header th:replace="~{common/header}"></header>

~~~

#### index.html에 레이아웃 적용

~~~html
<head th:replace="~{common/head :: head}"></head>
<header th:replace="~{common/header}"></header>
~~~

~~~html
~{디렉토리/파일명}
~~~

*링크 표현은 다음과 같다.*
~~~html
"@{/css/pagemain.css}"
~~~

로그인 여부에 따라 네비게이션 바
~~~html
<a th:if="${isLogin}" th:href="@{/logout}"><span>로그아웃</span></a>
<a th:unless="${isLogin}" th:href="@{/login}"><span>로그인 |</span></a>
~~~

로그인 여부를 ~isLogin~ 에 담아서 판단
~~~java
HttpSession session = request.getSession(false);
boolean isLogin = session != null && session.getAttribute(SessionConst.LOGIN_MEMBER) != null;
model.addAttribute("isLogin",isLogin);
~~~

*적용 index*

![](https://velog.velcdn.com/images/junified7/post/00eea611-0c33-4ab0-95ef-b548d335d9e6/image.png)

*회원가입 폼*

![](https://velog.velcdn.com/images/junified7/post/9ca31f04-ca2b-43cd-b8d5-52f390ff8501/image.png)

*수정 폼*
![](https://velog.velcdn.com/images/junified7/post/e831ebe0-ee43-4071-896d-e78a0b83b5dd/image.png)
*isLogin = true*
![](https://velog.velcdn.com/images/junified7/post/4cf3caae-2e9a-414c-9be5-b15a4d34ad16/image.png)<!-- {"width":473} -->
### 수정 시 id값을 url?

~~~html
<input type="hidden" th:field="*{id}">
~~~
값을 숨겨 **수정하려 접근한 id를 업데이트**

![](https://velog.velcdn.com/images/junified7/post/2d24fa72-4830-4819-8e04-54c54c1a5c5d/image.png)

하지만 사용자가 임의로 값을 변경할 수 있음

#### 해결방안

**숨겨진 필드 대신 서버 측 세션 사용**

~~~java
HttpSession session = request.getSession(false);
Member loginUser = (Member) session.getAttribute(SessionConst.LOGIN_MEMBER);
memberRepository.update(loginUser.getId(), member);
~~~

성공적으로 받아옴
![](https://velog.velcdn.com/images/junified7/post/df4df98a-52d4-4256-9401-a997c0dad613/image.png)

### 정리

**타임리프 속성으로 레이아웃 및 조건 설정**
- th:replace, th:include 같은 속성을 통해 HTML 템플릿의 재사용성을 높이는 레이아웃 설정을 지원한다.이를 통해 공통적인 페이지 구성 요소를 중복 없이 관리할 수 있다.
- 로그인 여부(isLogin)를 세션에서 확인하여 모델에 추가하고, 이를 타임리프 템플릿에서 th:if와 th:unless를 사용해 조건부 렌더링한다.

**ID 안전한 처리**
- 정보 수정 시 사용자가 임의로 ID 값을 변경할 수 있는 문제를 방지하기 위해, 숨겨진 필드 대신 서버 측 세션에서 로그인한 사용자의 ID를 직접 사용한다.
- 이 방식은 폼 데이터로부터 사용자 ID를 전달받지 않으므로, 사용자가 데이터를 조작할 가능성을 원천적으로 차단한다.
