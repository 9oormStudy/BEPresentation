## REST API 란 

<img width="573" alt="스크린샷 2024-04-11 오후 4 05 57" src="https://github.com/9oormStudy/BEPresentation/assets/53373279/b522aa76-d04f-47d1-a4aa-7a8c9d61375d">

- REST API 란? 서버의 자원을 클라이언트에 구애받지 않고 사용할 수 있게 하는 설계 방식

<img width="573" alt="스크린샷 2024-04-11 오후 4 05 57" src="https://github.com/9oormStudy/BEPresentation/assets/53373279/0412eef0-895b-4890-9a07-07b952902b3c">

- 뷰를 반환하도록 설계된 서버
- 다양한 클라이언트의 요청에 맞추어 어떤 기기가 와도 해당하는 뷰 페이지를 제공해야함
<img width="573" alt="스크린샷 2024-04-11 오후 4 05 57" src="https://github.com/9oormStudy/BEPresentation/assets/53373279/760bd035-5f2d-4583-910a-1d5e8d345fdf">

- REST API 로 설계된 서버
- 서버의 응답이 특정 기기에 종속되지 않도록 모든 기기에서 통용될 수 있는 데이터를 반환

  
### REST
- REST : 자원을 이름으로 구분하여 해당 자원의 상태를 주고받는 모든 것
- 자원을 이름으로 구분: URI 를 통해 자원의 이름을 구분한다.
- 자원의 상태를 주고 받는 모든 것: HTTP Method 를 통해 행위를 명시하고 해당 자원을 json, xml 과 같은 형태로 주고 받음

<img width="746" alt="스크린샷 2024-04-11 오후 4 40 01" src="https://github.com/9oormStudy/BEPresentation/assets/53373279/099e568f-b481-4e71-bac4-a91b6ea85a27">

### REST의 구성
- 자원: URI
  - 서버의 자원을 HTTP URI 를 통해 명시한다.
  - EX) 영화 정보가 서버의 자원일 때, /movie를 해당 자원의 표현으로 정한다.
- 행위: Http Method
  - GET, PUT, DELETE, PATCH 등 HTTP Method 를 통해 자원에 대한 CRUD (생성, 조회, 수정, 삭제) 를 적용
- 표현: json, xml
  - 클라이언트가 자원에 대한 조작을 요청하면 서버는 이에 적절한 응답을 보낸다.
  - 자원은 json, xml, text 등 여러가지 형태로 표현될 수 있다. 
  
즉, REST 는 HTTP URI 를 통해 Client 와 Server 사이에 통신하는 방식 중 하나로, HTTP Method 를 통해 자원을 처리 하도록 설계된 아키텍쳐를 말한다.   

### API
- API : 클라이언트가 서버의 자원을 요청할 수 있도록 서버에서 제공하는 인터페이스

### REST API
- REST API 란 결국 REST 를 기반으로 구현한 API

### RESTful API
<img width="517" alt="스크린샷 2024-04-11 오후 5 08 56" src="https://github.com/9oormStudy/BEPresentation/assets/53373279/3fe80cd3-3d87-4191-bfd2-2f78e0d217e1">

- RESTful 하다는 것은 REST 를 기반으로 잘 설계된 API 를 의미한다.
- RESTful 한 API 는 사진처럼 딱 봐도 어떠한 요청인지 한눈에 이해할 수 있다.

### REST API를 만드는 이유
- 과거에 클라이언트 프로그램의 대상은 PC 브라우저로 대상이 명확했다. 따라서 서버는 PC 브라우저에 적합한 웹 페이지를 구성하고 반환하면 됐다.
- 하지만 스마트 기기들이 등장하면서 TV, 스마트폰, 테블릿 등 클라이언트 프로그램이 다양화 되었고 그게 맞춰 server를 일일이 개발하는 것은 비효올 적이다.
- 이런 과정에서 기기에 종속되지 않은 일관된 응답을 위한 서버를 만들기 위해 REST API 로 설계된 서버를 만들게 되었다. 

### REST API URI 설계 규칙
1. URI 는 대문자보다는 소문자를 사용하여야 한다.
   - Bad Example http://khj93.com/getUsers/
   - Good Example  http://khj93.com/users/
     - 대문자가 문제를 일으킬 수 있기 때문
     - 1. http://api.example.com/my-folder/my-doc
       2. HTTP://API.EXAMPLE.COM/my-folder/my-doc
     - 위 두개 URI 는 같은 경로로 인식하지만
     - http://api.example.com/My-Folder/my-doc
     - 이 URI 는 a, b 와 같지 않다.

2. 마지막에 슬래시 (/)를 포함하지 않는다.
   - Bad Example http://khj93.com/test/
   - Good Example  http://khj93.com/test
      - 많은 브라우저와 프레임워크에서 두 URI를 같다고 인식한다.
      - 서로 다른 URI 는 다른 자원을 나타내야 하기 때문에 후행 / 는 혼란을 야기한다. 

3. 언더바 대신 하이폰을 사용한다.
   - Bad Example http://khj93.com/test_blog
   - Good Example  http://khj93.com/test-blog
     - 가독성을 위함
     - '-' 대신 '_' 같은 경우는 폰트로 인해 보이지 않거나 하이퍼 링크에서 보이지 않을 수 있음
     - http://khj93.com/test__blog

4. 파일확장자는 URI에 포함하지 않는다.
   - Bad Example http://khj93.com/photo.jpg
   - Good Example  http://khj93.com/photo
     - 파일의 확장자는 header 를 통해 전달되는 media-type에 의존 하여야 한다. 

5. 행위를 포함하지 않는다.
   - Bad Example /chatrooms/create
   - Good Example  /chatrooms
     - 행위는 URI 가 아닌 Method 를 통해 표현 되어야 한다. 
   
