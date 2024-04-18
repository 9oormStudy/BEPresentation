**Web이란?**

인터넷에 연결된 사용자들이 서로의 정보를 공유할 수 있는 공간을 말한다.

(그것을 간단히 줄여서 우리가 아는 WWW나 W3라고도 불르며, 흔하게 웹(Web)이라고 많이 불린다.)

인터넷과 같은 의미로 많이 사용되지만

정확하게 웹(Web) 과 인터넷은 같은 의미가 아니다.

웹(Web)은 인터넷상 하나의 서비스중 하나일 뿐입니다.

(하지만 서로 혼용되어 사용할 만큼 인터넷에서 가장 큰 부분을 차지하고 있다.)

- 1세대 웹 서비스
    
    1세대 웹사이트는 모두 단순하고, 정적(static)인 웹사이트였습니다.
    
    원하는 자료를 찾는것만 허용되는 사이트였다. (HTML)
    
    최초의 웹 사이트 출저 : http://info.cern.ch/
    

- 2세대 웹 서비스
    
    넷스케이프사에서 JavaScript를 개발하여, JavaScript를 활용한 동적인 브라우저가 처음 생겨났지만 여전히 HTML , CSS가 주를 이룹니다.
    
    (이때 글을 쓰거나 수정하는 컨텐츠가 생겨나기 시작합니다.)
    
- 3세대 웹 서비스
    
    동적인 요소가 중요시되며 HTML,CSS보다 JavaScript 위주로 코드가 작성되기 시작합니다.
    
    (점점 검색플랫폼, 쇼핑몰, 유튜브 등이 자리를 유명해집니다.
    
    또한 프론트와 백엔드가 구분되는 시기)
  
![웹](https://github.com/9oormStudy/BEPresentation/assets/109911154/96d27ef8-e5ba-46a8-aebc-959b2a03d014)

WWW의 요소들

1. 웹브라우저(Web Browser)
    
    정보(문서,사진,동영상)을 주고 받을 수 있는 도구
    
2. 웹서버(Web Server)
    
    웹 브라우저로부터 받은 요청을 제공
    
3. HTTP
    
    HyperText Transfer Protocol의 약자로 하이퍼텍스트(HTML)를 빠르게 교환하기 위한 통신으로,
    
    HTTP는 서버와 클라이언트의 사이에서 어떻게 메세지를 교환할 지를 정해 놓은 규칙인 것이다.
    
4. HTML
    
    웹 브라우저 화면에 출력 되는 정보를 문서화,사진화,동영상으로 만들어주는 HTML


Web 서비스 동작과정


![웹통신](https://github.com/9oormStudy/BEPresentation/assets/109911154/1846e16f-7a3b-4d64-8188-1b84a1769070)


1. 웹 브라우저와 웹서버가 HTTP를 이용하여 통신한다.
    
    
2. 웹 브라우저에서 특정 URL을 웹서버에 요청한다.
    
    
3. 웹 서버는 해당 디렉토리에서 특정 HTML 파일을 찾아서 다시 웹 브라우저에게 제공한다.
    
    
4. 웹브라우저는 제공받은 HTML파일을 클라이언트 화면에 띄어준다.


마지막으로..

**웹 개발 관련 직군**

- `기획자 (Product Manager)`
    
    개발하려는 서비스를 정의하고 기획한다.
    
- `디자이너 (Designer)`
    
    `UI 및 UX`를 구현하는 역할을 담당한다.
    
    사용자가 사용하기 편리하게, 눈에 보기 좋게 디자인하는 역할이다.
    
- `프론트엔드 개발자 (Frontend Developer)`
    
    `HTML, CSS, JavaScript`로 프론트엔드 시스템을 구현한다.
    
    사용자와 가장 가까이 연결된 개발자이다.
    
- `백엔드 개발자 (Backend Developer)`
    
    백엔드 시스템을 개발하며 크게 2가지로 나뉜다.
    
    백엔드에서 앞 쪽(API)을 담당하는 개발자
    
    백엔드의 뒤쪽 - 데이터 수집,분석,관리 등 데이터 관련 시스템을 개발하는 개발자
    
- `DevOps (Development Operations)`
    
    시스템 개발과 시스템 운영을 함께 담당한다. 직군이라기 보다는 개발 문화의 추세이다.
    
    백엔드 개발자가 서버 구축 및 운영 등 System infrastructure 관리까지 담당하는 추세이다.
    
- `SysOps (System Operations)`
    
    System Infrastructure의 구현과 관리, 운영을 담당하는 직군으로 DevOps와 다르게 실제 하드웨어를 다룰 수 있다.
    
    Data Center를 사용해 시스템을 운영하는 회사에 필요하지만, AWS같은 클라우드 서비스가 보편화되며 DevOps 개발자들이 System Infrastructure를 담당하는 추세에 따라 사라지고 있는 직군이다.
    
- `Data Scientist`
    
    Machine Learning, AI 등을 이용해 많은 양의 데이터를 분석해 새로운 정보와 가치로 만들어내는 직군이다.
    
- `Data Engineer`
    
    Data Scientist가 데이터를 분석할 수 있도록 데이터를 정리하는 시스템을 구현하는 역할이다.
    
- `Tester`
    
    시스템을 테스트하여 검증하는 직군이다.
    
    직접 manual testing 하는 QA (Quality Assurance), 자동 테스트 시스템을 개발하는 Software Engineer in Test / Test Automation Engineer 등이 있다.
