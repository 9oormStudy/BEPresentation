## MVC 프레임워크 만들기

## 프론트 컨트롤러 도입 - V1 
### 변경점
- 프론트 컨트롤러에서 모든 요청을 먼저 받음 
- controllerMap 도입
- HttpServletRequest 객체를 Model 로 사용

### 실행 순서
1. 컨트롤러 호출
2. `FrontController` service() 실행
3. `FrontController` controllerMap 에서 컨트롤러 찾아서 호출
4. `Controller` request 객체에 데이터 보관
5. `Controller` forward() 로 뷰 호출 
### FrontControllerServletV1
```java
//urlPatterns = "/front-controller/v1/*" : "/front-controller/v1" 을 포함한 모든 하위 요청을 먼저 받음 
@WebServlet(name = "frontControllerServletV1", urlPatterns = "/front-controller/v1/*")
public class FrontControllerServletV1 extends HttpServlet {

    private Map<String, ControllerV1> controllerMap = new HashMap<>();

    public FrontControllerServletV1() {
        controllerMap.put("/front-controller/v1/members/new-form", new MemberFormControllerV1());
        controllerMap.put("/front-controller/v1/members/save", new MemberSaveControllerV1());
        controllerMap.put("/front-controller/v1/members", new MemberListControllerV1());
    }

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("FrontControllerServletV1.service");

        String requestURI = request.getRequestURI();

        ControllerV1 controller = controllerMap.get(requestURI);
        if (controller == null) {
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }

        controller.process(request, response);
    }
}
```
- urlPatterns = "/front-controller/v1/*" 는 "/front-controller/v1" 을 포함한 모든 하위 요청을 받게 합니다.
- controllerMap 에 url을 저장하고 요청 URI를 기반으로 컨트롤러를 찾아서 반환하고 실행
  
### MemberSaveControllerV1
```java
public class MemberSaveControllerV1 implements ControllerV1 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    public void process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        //Model에 데이터를 보관한다.
        request.setAttribute("member", member);

        String viewPath = "/WEB-INF/views/save-result.jsp";
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
    }
}
```
- HttpServletRequest 객체를 Model 로 사용해서 member 객체를 저장
- Controller 에서 직접 View 로 이동하는 forward() 로직을 실행 
 



## View 분리 - V2 
### V1 의 문제점
- 뷰로 이동하는 forward() 로직이 모든 컨트롤러에 중복되어있다.

### 변경점
- forward() 로직의 중복을 제거하기 위해 MyView 객체 도입

### 실행 순서
1. 프론트 컨트롤러 호출
2. `FrontController` service() 실행
3. `FrontController` controllerMap 에서 컨트롤러 찾아서 호출
4. `Controller` request 객체에 데이터 보관
5. `Controller` 컨트롤러는 바로 forwoard() 로직을 실행하지 않고 MyView 객체에 viewPath 를 담아서 반환
6. `FrontController` 반환받은 MyView 객체의 render() 실행
7. `MyView` forward() 로직 실행 

### FrontControllerServletV2
```java
@WebServlet(name = "frontControllerServletV2", urlPatterns = "/front-controller/v2/*")
public class FrontControllerServletV2 extends HttpServlet {

    private Map<String, ControllerV2> controllerMap = new HashMap<>();

    public FrontControllerServletV2() {
        controllerMap.put("/front-controller/v2/members/new-form", new MemberFormControllerV2());
        controllerMap.put("/front-controller/v2/members/save", new MemberSaveControllerV2());
        controllerMap.put("/front-controller/v2/members", new MemberListControllerV2());
    }

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        String requestURI = request.getRequestURI();

        ControllerV2 controller = controllerMap.get(requestURI);
        if (controller == null) {
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }

        MyView view = controller.process(request, response);
        view.render(request, response);
    }
}
```

### MemberSaveControllerV2
```java
public class MemberSaveControllerV2 implements ControllerV2 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    public MyView process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        //Model에 데이터를 보관한다.
        request.setAttribute("member", member);

        return new MyView("/WEB-INF/views/save-result.jsp");
    }
}
```
-맨 아래 3개 라인에서 View 로 이동하는 부분이 중복됨 -> MyView 객체 도입으로 중복 해결 

### MyView
```java
public class MyView {

    private String viewPath;

    public MyView(String viewPath) {
        this.viewPath = viewPath;
    }

    public void render(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
    }
}
```
- 컨트롤러는 forward() 로직을 실행하지 않고 MyView 객체를 생성하면서 viewPath 를 전달하고 forward() 로직을 MyView 객체에 위임

## Model 추가 - V3 
### V2 의 문제점 
- 컨트롤러 입장에서 HttpServletRequest 와 같은 서블릿 기술은 알 필요가 없다.

### 변경점
- ModelView 객체 추가: HttpServletRequest 객체를 Model 로 사용하는 대신에 별도의 Model 객체를 만들어서 사용한다. 
- ViewResolver 추가: 뷰의 논리이름을 반환하고 실제 이름은 프론트 컨트롤러에서 처리하도록 한다. 
- createParamMap 함수 추가 : HttpServletRequest 객체를 사용하지 않기 때문에 paramMap 객체에 저장하고 컨트롤러로 전달 
- modelToRequestAttribute 메서드 추가 : MyView 로 전달하는 model 객체는 단순히 Map 이기 때문에 request 객체에 저장하는 함수 추가. 

### 실행 순서                         
1. 프론트 컨트롤러 호출
2. `FrontController` service() 실행
3. `FrontController` request 객체의 모든 파라미터를 paramMap에 저장 (Controller 에서 HttpServletRequest 객체를 사용하지 않기 때문) 
4. `FrontController` controllerMap 에서 컨트롤러 찾아서 호출 (파라미터로 paramMap 전달)
5. `Controller` 컨트롤러에서 ModelView 객체에 뷰의 논리이름을 저장하고 ModelView 객체에서 model을 받아서 member 객체를 저장
6. `FrontController` 반환받은 ModelView 객체에서 논리이름을 받고 viewResolver() 로 실제 주소로 변환
7. `FrontController` MyView 객체의 render() 함수를 실행 파라미터로 model 객체를 넘김
8. `MyView` 전달받은 model 객체는 단순한 map 이므로 request 객체에 setAttribute로 저장
9. `MyView` forward() 로직 실행

### FrontControllerServletV3
```java
@WebServlet(name = "frontControllerServletV3", urlPatterns = "/front-controller/v3/*")
public class FrontControllerServletV3 extends HttpServlet {

    //paramMap 생략

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        //controller 찾기 생략

        //request 객체의 모든 파라미터를 paramMap 객체에 저장한다. (paramMap 객체를 request 객체처럼 사용할 수 있음)
        Map<String, String> paramMap = createParamMap(request);

        ModelView mv = controller.process(paramMap);

        String viewName = mv.getViewName();
        MyView view = viewResolver(viewName);

        view.render(mv.getModel(), request, response);
    }

    private MyView viewResolver(String viewName) {
        return new MyView("/WEB-INF/views/" + viewName + ".jsp");
    }

    private Map<String, String> createParamMap(HttpServletRequest request) {
        Map<String, String> paramMap = new HashMap<>();
        request.getParameterNames().asIterator()
                .forEachRemaining(paramName -> paramMap.put(paramName, request.getParameter(paramName)));
        return paramMap;
    }
}
```
### MemberSaveControllerV3
```java
public class MemberSaveControllerV3 implements ControllerV3 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    public ModelView process(Map<String, String> paramMap) {
        String username = paramMap.get("username");
        int age = Integer.parseInt(paramMap.get("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        ModelView mv = new ModelView("save-result");
        mv.getModel().put("member", member);
        return mv;
    }
}
```

### MyView
```java
public class MyView {

    private String viewPath;

    public MyView(String viewPath) {
        this.viewPath = viewPath;
    }

    public void render(Map<String, Object> model, HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        modelToRequestAttribute(model, request);
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
    }

    private void modelToRequestAttribute(Map<String, Object> model, HttpServletRequest request) {
        model.forEach((key, value) -> request.setAttribute(key, value));
    }
}
```

## 단순하고 실용적인 컨트롤러 - V4 
### V3 문제점
- ModelView 를 매번 생성해야함

### 변경점
- Controller 에서 ViewName 만 반환: ModelView 를 매번 생성해야 하는 번거로움을 개선
- 프론트 컨트롤러에서 model 객체를 생성하서 controller 에 파라미터로 전달
- ModelView 객체의 model을 사용하지 않음 

### FrontControllerServletV4
``` java
@WebServlet(name = "frontControllerServletV4", urlPatterns = "/front-controller/v4/*")
public class FrontControllerServletV4 extends HttpServlet {

    //paramMap 생략

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        String requestURI = request.getRequestURI();

        ControllerV4 controller = controllerMap.get(requestURI);
        if (controller == null) {
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }

        Map<String, String> paramMap = createParamMap(request);
        Map<String, Object> model = new HashMap<>(); //추가

        String viewName = controller.process(paramMap, model);

        MyView view = viewResolver(viewName);
        view.render(model, request, response);
    }
 }
```
### MemberSaveControllerV4
```java
public class MemberSaveControllerV4 implements ControllerV4 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    public String process(Map<String, String> paramMap, Map<String, Object> model) {
        String username = paramMap.get("username");
        int age = Integer.parseInt(paramMap.get("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        model.put("member", member);
        return "save-result";
    }
}
```

## Spring Controller 

```java
    @PostMapping("/add")
    public String addItemV1(@RequestParam String itemName,
                       @RequestParam int price,
                       @RequestParam Integer quantity,
                       Model model) {

        Item item = new Item();
        item.setItemName(itemName);
        item.setPrice(price);
        item.setQuantity(quantity);

        itemRepository.save(item);

        model.addAttribute("item", item);

        return "basic/item";
    }
```
- 파라미터에 Model 객체가 있으면 스프링이 자동으로 Model 객체를 생성해서 파라미터로 넘겨줌 
```java
    @PostMapping("/add")
    public String addItemV2(@ModelAttribute("item") Item item, Model model) {

        itemRepository.save(item);
        model.addAttribute("item", item); //자동 추가, 생략 가능

        return "basic/item";
    }
```
- @ModelAttribute 로 Item 객체 생성 및 저장과정 생략 
``` java
    @PostMapping("/add")
    public String addItemV3(@ModelAttribute Item item) {
        itemRepository.save(item);
        return "basic/item";
    }
```
- 파라미터에 Model 객체와 model.addAttrubute() 도 생략 가능
- return "basic/item" : 뷰 이름을 반환하면 ViewResolver 가 동작해서 뷰 생성