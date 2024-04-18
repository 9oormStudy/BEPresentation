# 빈 스코프

- 싱글톤
- 프로토타입
- 웹 관련 스코프

### 싱글톤

> 가본 스코프로, 스프링 IoC 컨테이너 당 빈 인스턴스가 하나만 생성  
컨테이너의 시작과 종료까지 유지되는 스코프

#### 사용사례 

> 싱글톤 스코프는 상태를 유지하지 핞는 서비스, 레포지토리, 헬퍼 클래스 등에서 사용

#### 특징

- 이 스코프는 메모리 사용량을 최소화하는 데 도움이 됨.
- 모든 클라이언트가 같은 인스턴스를 공유하므로 상태를 유지하면 안 됨.

### 프로토타입

> 빈에 대한 요청이 있을 때마다 새로운 인스턴스 생성

#### 사용사례

> 각 클라이언트가 고유한 인스턴스를 가지길 원할 때 사용한다.  
예를 들어, 사용자별 상태 정보를 조장하는 컴포넌트

#### 특징

- 빈의 생성과 초기화 비용이 클 수 있음
- 스프링 컨테이너는 인스턴스의 전체 생명주기를 관리하지 않는다. 즉, 생성 후엔 클라이언트가 관리해야 함
- 종료 메서드가 호출되지 않음

### 리퀘스트

> 각 http 요청마다 빈 인스턴스가 하나씩 생성

#### 사용사례

> 앱 애플리케이션에 사용자 요청마다 상태를 유지해야 하는 컴포넌트에 사용

#### 특징

- 이 스코프는 웹 애플리케이션에만 유효
- 각 요청은 고유한 빈 인스턴스를 가지게 됨

### 세션

> 각 http 세션마다 빈 인스턴스 하나씩 생성

#### 사용사례

> 로그인한 사용자의 정보를 저장하는 컴포넌트에서 사용

#### 특징

- 이 스코프는 웹 애플리케이션에서만 유효
- 사용자 별 상태 정보를 유지할 수 있음

## 싱글톤 빈과 프로토타입 스코프를 같이 사용?

> 싱글톤 빈이 프로토타입 빈을 주입받게 된다면, 싱글톤 빈이 생성될 때 단 한번만 프로토타입 빈이 생성되어 주입된다.  
이후에 싱글톤 빈이 계속해서 같은 프로토타입 빈 인스턴스를 사용하게 된다.  
따라서 프로토타입 스코프의 의도와 다르게 매번 새로운 인스턴스가 생성되지 않는 문제가 발생

### 해결 방안

#### ObjectProvider<T>

> 이 문제를 해결하기 위해 스프링은 ObjectProvider<T> 인터페이스 제공

~~~java
@Component
public class SingletonBean {

    @Autowired
    private ObjectProvider<PrototypeBean> prototypeBeanProvider;

    public void action() {
        PrototypeBean prototypeBean = prototypeBeanProvider.getObject();
        // 이 시점에 새로운 PrototypeBean 인스턴스가 생성됩니다.
    }
}
~~~

#### 장점

1. 지연 로딩 : 실제로 필요한 시점까지 객체 생성을 지연시킬 수 있어, 애플리케이션의 시작 시간을 단축, 리소스 사용 최적화 가능
2. 유연성 : 메소드를 호출하는 시점에 객체를 요청함으로 런타임에 동적으로 빈 관리할 수 있는 유연성 제공 
3. 프로토타입 스코프 관리 : 싱글톤 빈이 프로토타입 빈을 참조하는 대신, *objectProvider*를 통해 필요할 때마다 새로운 인스턴스 제공

#### @Lookup

> *@Lookup* 애노테이션이 달린 메소드가 호출될 때 스프링이 bean을 찾아 리턴해준다.

~~~java
@Service
class CommonService {
    @Lookup
    fun getNotification(): Notification? {
        return null
    }
}
~~~

#### 프록시

> 스프링엔 프록시를 사용해 빈의 실제 인스턴스 대신 프록시 객체를 빈으로 등록하고, 해당 프록시 객체를 통해 대상 빈에 접근하도록 한다.

**프록시는 AOP와 함께 스프링 애플리케이션의 다양한 부분에 활용됨**

~~~java
  // 적용 대상이 클래스면 TARGET_CLASS, 인터페이스면 INTERFACES
@Component
@Scope(value = "prototype", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class PrototypeBean {
    // 프로토타입 빈의 구현
}

@Component
public class SingletonBean {
    private final PrototypeBean prototypeBean;

    @Autowired
    public SingletonBean(PrototypeBean prototypeBean) {
        this.prototypeBean = prototypeBean;
    }

    public void action() {
        // prototypeBean을 사용하는 로직
    }
}
~~~

## 마무리

### 빈 스코프

- 스프링은 다양한 빈 스코프를 제공, 애플리케이션의 요구 사항에 맞게 빈의 생명주기 관리할 수 있도록 함.
- 싱글톤은 애플리케이션 내 하나의 인스턴스만 유지, 프로토타입은 요청마다 인스턴스 생성
- 웹 관련은 요청의 생명주기에 맞춰 빈의 생명주기 관리

### 싱글톤, 프로토타입 같이 사용

- ObjectProvider<T>, @Lookup 애노테이션, 프록시 방식을 사용
- ObjectProvider<T>, @Lookup은 요청 시마다 새로운 프로토타입 빈 인스턴스를 제공받는 방법
- 프록시 방식은 스프링이 프록시 객체를 통해 자동으로 새로운 프로토타입 빈 인스턴스를 제공하도록 하는 방법

### 정리

- 애플리케이션의 특성과 요구 사항에 맞게 적절한 빈 스코프를 선택하는 것이 중요
- 싱글톤, 프로토타입을 혼합할 땐, 스프링이 제공하는 해결 방법 활용