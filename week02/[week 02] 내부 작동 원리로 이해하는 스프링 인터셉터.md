# 내부 작동 원리로 이해하는 스프링 인터셉터

스프링 인터셉터는 모든 요청을 처리하는 디스패처 서블릿(DispatcherServlet) 의 doDispatch 메서드에서 아래와 같은 구조로 컨트롤러 호출 전, 후 로 실행하는 구조로 되어있습니다.

`DispatcherServlet` 클래스의 doDispatch 메소드
```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {

    ...
    
    if (!mappedHandler.applyPreHandle(processedRequest, response)) {
        return; // preHandle에서 false 반환 시 여기서 처리 중단
    }
    
    ...
    
    try {
      try {
          // 핸들러 어댑터를 통해 컨트롤러 메서드 호출
          mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
          // 예외 발생시 catch 로 인해 postHandle 요청 안됨
          // postHandle 호출
          mappedHandler.applyPostHandle(processedRequest, response, mv); 
      } catch (Exception ex) {
          dispatchException = ex; // 예외 발생 시 저장
      }
      // afterCompletion은 예외 발생 유무와 상관없이 항상 호출
      this.processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
    }
    
    ...

}

protected void processDispatchResult(HttpServletRequest request, HttpServletResponse response, HandlerExecutionChain mappedHandler, ModelAndView mv, Exception ex) throws Exception {
    // 정상 완료 또는 예외 발생 시 afterCompletion 호출
    if (ex != null) {
        if (mappedHandler != null) {
            mappedHandler.triggerAfterCompletion(request, response, ex);
        }
    } else {
        // 정상 흐름에서의 afterCompletion 호출
        mappedHandler.applyAfterCompletion(request, response, null);
    }
}
```

doDispatch 메서드 안의  preHandle 메서드로 개발자가 등록한 인터셉터의 preHandle 이 실행되는 과정은 다음과 같습니다.

#### 1. 인터셉터 등록

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LogInterceptor()) // 인터셉터를 등록
                .order(1) // 호출 순서 지정 (낮을 수록 먼저 호출)
                .addPathPatterns("/**") // 인터셉터 적용할 URL 패턴 지정
                .excludePathPatterns("/css/**", "/*.ico", "/error"); // 인터셉터에서제외할 패턴 지정
    }
}
```

addInterceptors 메소드를 통해 InterceptorRegistry에 인터셉터를 등록합니다. 이때 LogInterceptor 인스턴스를 생성하고, 호출 순서, 적용할 URL 패턴, 제외할 URL 패턴을 설정합니다. 이 정보는 스프링 MVC의 내부 구성에 추가됩니다.

#### 2. HandlerExecutionChain 객체

```java
protected void doDispatch(...) throws Exception {

    HandlerExecutionChain mappedHandler = null;

    mappedHandler = this.getHandler(processedRequest);

}
```

핸들러 매핑 과정에서, 스프링 MVC는 요청에 해당하는 핸들러(컨트롤러 메소드)뿐만 아니라, 이 요청에 적용될 인터셉터 목록을 함께 결정합니다. 이때, WebConfig에서 설정한 인터셉터들이 포함된 HandlerExecutionChain 객체가 생성됩니다. 생성된 HandlerExecutionChain 객체는 요청에 맞는 핸들러와 그 핸들러에 적용될 모든 인터셉터들의 정보를 포함합니다.


#### 3. applyPreHandle 메소드

```java
    boolean applyPreHandle(HttpServletRequest request, HttpServletResponse response) throws Exception {
        for(int i = 0; i < this.interceptorList.size(); this.interceptorIndex = i++) {
            HandlerInterceptor interceptor = (HandlerInterceptor)this.interceptorList.get(i);
            if (!interceptor.preHandle(request, response, this.handler)) {
                this.triggerAfterCompletion(request, response, (Exception)null);
                return false;
            }
        }

        return true;
    }
```

applyPreHandle 메소드는 HandlerExecutionChain에 등록된 인터셉터 목록을 순서대로 처리합니다. 이 때, 각 인터셉터의 preHandle 메소드가 차례대로 호출됩니다.

여기서 this.interceptors는 HandlerExecutionChain에 등록된 인터셉터 목록을 나타내고, this.handler는 실제 요청을 처리할 핸들러(컨트롤러)를 가리킵니다. 

이처럼 applyPreHandle은 등록된 인터셉터들을 순회하며 각각의 preHandle을 호출하여 요청 전 처리를 수행합니다.

 나머지 postHandle과 afterCompletion 메소드들도 비슷한 과정을 통해 이해할 수 있습니다. 
