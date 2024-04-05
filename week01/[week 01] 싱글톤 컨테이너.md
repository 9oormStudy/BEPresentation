## 싱글톤 컨테이너

### 웹 애플리케이션과 싱글톤
- 스프링은 태생이 기업용(enterprise) 온라인 서비스 기술을 지원하기 위해 탄생했다. 
- 웹 애플리케이션은 보통 여러 고객이 동시에 요청을 한다.
![img](https://velog.velcdn.com/images%2Fsyleemk%2Fpost%2F5d6467ac-9ba3-49a2-9d5c-0a7df8385f0c%2Fimage.png)  
- AppConfig의 경우 요청을 할 때 마다 객체를 새로 생성한다.
- 이는 메모리 낭비가 심하여 해당 객체를 딱 1개만 생성하고 공유하도록 설계하는데 이를 **싱글톤 패턴**이라 한다.

##### 싱글톤 패턴
- 클래스의 인스턴스가 딱 1개만 생성되는 것을 보장하는 디자인 패턴이다.
- 때문에 객체 인스턴스를 2개 이상 생성하지 못하도록 막아야 함
    - private 생성자를 사용해서 외부에서 임의로 new 키워드를 사용하지 못하도록 막아야 한다.


#### 싱글톤 패턴 예제 

```
//자바스크립트의 싱글톤 패턴
const obj = {
    a: 27
}
const obj2 = {
    a: 27
}
console.log(obj === obj2)
// false -> obj와 obj2는 다른 인스턴스를 가진다
```
```
// Singleton.instance라는 하나의 인스턴스를 가지는 Singleton 클래스
class Singleton {
    constructor() {
        if (!Singleton.instance) {
            Singleton.instance = this
        }
        return Singleton.instance
    }
    getInstance() {
        return this 
    }
}
const a = new Singleton()
const b = new Singleton() 
console.log(a === b) // true a와 b는 하나의 인스턴스
```
#### 강의에 나온 코드

```
package hello.core.singleton;
public class SingletonService {
 //1. static 영역에 객체를 딱 1개만 생성해둔다.
 private static final SingletonService instance = new SingletonService();
 //2. public으로 열어서 객체 인스턴스가 필요하면 이 static 메서드를 통해서만 조회하도록 허용한
다.
 public static SingletonService getInstance() {
 return instance;
 }
 //3. 생성자를 private으로 선언해서 외부에서 new 키워드를 사용한 객체 생성을 못하게 막는다.
 private SingletonService() {
 }
 public void logic() {
 System.out.println("싱글톤 객체 로직 호출");
 }
}
```
> 참고 : 싱글톤 패턴을 구현하는 방법은 이외에도 여러가지가 있지만, 위의 코드는 가장 단순하고 안전한 객체를 미리 생성하는 방식을 택함

### 싱글톤 패턴의 문제점
- 싱글톤 패턴을 구현하는 코드 자체가 많이 들어간다.
- 의존 관계 상 클라이언트가 구체 클래스에 의존한다. -> DIP를 위반
- 클라이언트가 구체 클래스에 의존해서 OCP 원칙을 위반할 가능성이 높다.
- 테스트하기 어렵다.
- 내부 속성을 변경하거나 초기화 하기 어렵다.
- private 생성자로 자식 클래스를 만들기 어렵다.
- 결론적으로 유연성이 떨어진다.
- 안티패턴으로 불리기도 한다.  
  

> 안티패턴(anti-pattern) : 실제 많이 사용되는 패턴이지만 비효율적이거나 비생산적인 패턴을 의미한다.
> 출처 : 위키백과 '안티패턴'

### 싱글톤 컨테이너
스프링의 싱글톤 컨테이너는 싱글톤 패턴의 문제점을 해결하면서 인스턴스를 싱글톤(인스턴스 1개만 생성)으로 관리한다. 

#### 싱글톤 컨테이너
- 스프링 컨테이너는 싱글톤 패턴을 적용하지 않아도 객체 인스턴스를 싱글톤으로 관리한다.
- 스프링 컨테이너는 알아서 싱글톤 컨테이너 역할을 한다. 이렇게 싱글톤 객체를 생성하고 관리하는 기능을 싱글톤 레지스트리라 한다.
- 스프링 컨테이너의 이런 기능 덕분에 싱글톤 패턴의 모든 단점을 해결하면서 객체를 싱글톤으로 유지할 수 있다.
    - 싱글톤 패턴을 위한 지저분한 코드가 들어가지 않아도 된다.
    - DIP, OCP, 테스트, private 생성자로부터 자유롭게 싱글톤을 사용할 수 있다.
```
// MemberService 생성자를 직접 호출하지 않음을 볼 수 있다.
void springContainer() {
    ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

    //1. 조회: 호출할 때 마다 같은 객체를 반환
    MemberService memberService1 = ac.getBean("memberService", MemberService.class);

    //2. 조회: 호출할 때 마다 같은 객체를 반환
    MemberService memberService2 = ac.getBean("memberService", MemberService.class);

    //참조값이 같은 것을 확인
    System.out.println("memberService1 = " + memberService1); // memberService1 = hello.core.member.MemberServiceImpl@61f05988
    System.out.println("memberService2 = " + memberService2); // memberService2 = hello.core.member.MemberServiceImpl@61f05988

    //memberService1 == memberService2 테스트. 무사히 통과됨
    assertThat(memberService1).isSameAs(memberService2); 
}
```

#### 싱글톤 컨테이너 적용 후
![img](https://velog.velcdn.com/images%2Fsyleemk%2Fpost%2Fb0c31f8b-8332-4389-af62-f7e26ccf1ce4%2Fimage.png)  
- 스프링 컨테이너 덕분에 고객의 요청이 올 때 마다 객체를 생성하는 것이 아니라, 이미 만들어진 객체를 공유해서 효율적으로 재사용할 수 있다.

> 참고 : 스프링의 기본 빈 등록 방식은 싱글톤이지만, 싱글톤 방식만 지원하는 것은 아니다. 요청할 때마다 새로운 객체를 생성해서 반환하는 기능도 제공한다.

### 싱글톤 방식의 주의점
- 싱글톤 패턴이든, 스프링 같은 싱글톤 컨테이너를 사용하든, 객체 인스턴스를 하나만 생성해서 공유하는 싱글톤 방식은 여러 클라이언트가 하나의 같은 객체 인스턴스를 공유하기 때문에 싱글톤 객체는 상태를 유지(stateful)하게 설계하면 안된다.
-  **무상태(stateless)** 로 설계해야 한다
    - 특정 클라이언트에 의존적인 필드가 있으면 안된다.
    - 특정 클라이언트가 값을 변경할 수 있는 필드가 있으면 안된다.
    - 가급적 읽기만 가능해야 한다.
    - 필드 대신 자바에서 공유되지 않는 지역변수, 파라미터, ThreadLocal 등을 사용해야 한다.

#### 상태를 유지할 경우 발생하는 문제점 예시
```

package hello.core.singleton;

public class StatefulService {
    
    private int price; // 상태를 유지하는 필드
    
    public void order(String name, int price){
        System.out.println("name = " + name + "price = " + price );
        this.price = price; // 여기가 문제! CS에서 critical section 처럼 공유자원에 접근
    }
    
    public int getPrice() {
        return price;
    }
}
```
#### 상태를 유지할 경우 발생하는 문제점 예시 Test
```
package hello.core.singleton;

import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;

import static org.junit.jupiter.api.Assertions.*;

class StatefulServiceTest {

    @Test
    void statefulServiceSingleton() {
        // 스프링 컨테이너 생성 및 statefulService 객체 호출
        ApplicationContext ac = new AnnotationConfigApplicationContext(TestConfig.class);
        StatefulService statefulService1 = ac.getBean("statefulService", StatefulService.class);
        StatefulService statefulService2 = ac.getBean("statefulService", StatefulService.class);

        // ThreadA : A사용자 10000원 주문
        statefulService1.order("userA", 10000);
        // ThreadB : B사용자 20000원 주문
        statefulService2.order("userB", 20000);

        // ThreadA : A사용자 주문 금액 조회
        int price = statefulService1.getPrice();
        System.out.println("priceA = " + price);

        Assertions.assertThat(statefulService1.getPrice()).isEqualTo(20000);

    }

    // Test를 위해 생성한 configuration class
    static class TestConfig{
        @Bean
        public StatefulService statefulService() {
            return new StatefulService();
        }
    }
}
```
- ThreadA가 사용자A 코드를 호출하고 ThreadB가 사용자B 코드를 호출한다 가정하면
- StatefulService 의 price 필드는 공유되는 필드인데,
특정 클라이언트가 값을 변경한다 (CS에서 critical section과 비슷하다고 느낌)
- 사용자A의 주문금액은 10000원이 되어야 하는데, 20000원이라는 결과가 나옴

### @Configuration과 싱글톤
- AppConfig 내에 있는 객체가 서로 관계를 맺고 호출되는 경우 동일한 객체를 여러 번 호출하는 경우가 발생한다.
- 결과적으로 같은 클래스로부터 각각의 다른 객체가 구현되면서 싱글톤이 깨지는 것 처럼 보이게 된다. 아래 코드를 보자.
```
package hello.core;
import hello.core.discount.DiscountPolicy;
import hello.core.discount.RateDiscountPolicy;
import hello.core.member.MemberRepository;
import hello.core.member.MemberService;
import hello.core.member.MemberServiceImpl;
import hello.core.member.MemoryMemberRepository;
import hello.core.order.OrderService;
import hello.core.order.OrderServiceImpl;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
@Configuration
public class AppConfig {
 @Bean
 public MemberService memberService() {
 //1번
 System.out.println("call AppConfig.memberService");
 return new MemberServiceImpl(memberRepository());
 }
 @Bean
 public OrderService orderService() {
 //1번
 System.out.println("call AppConfig.orderService");
 return new OrderServiceImpl(
 memberRepository(),
 discountPolicy());
 }
 @Bean
 public MemberRepository memberRepository() {
 //2번? 3번?
 System.out.println("call AppConfig.memberRepository");
 return new MemoryMemberRepository();
 }
 @Bean
 public DiscountPolicy discountPolicy() {
 return new RateDiscountPolicy();
 }
}
```
```
// 위 AppConfig를 호출하는 Test 코드
package hello.core.singleton;

import hello.core.AppConfig;
import hello.core.member.MemberService;
import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

import static org.assertj.core.api.Assertions.*;

public class singletonTest {
    @Test
    @DisplayName("스프링 컨테이너와 싱글톤")
    void springContainer() {
        // 스프링 컨테이너 생성(AppConfig.class 내의 @Bean 등록된 class들을 객체화 하여 컨테이너에 저장한다.)
        ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
}
}
```
- 스프링 컨테이너가 각각 @Bean을 호출하여 스프링 빈을 생성한다. 아마도  memberRepository()는 총 3번 출력될 것이다
- 그러나 출력 결과 모두 1번만 호출된다. 
```
call AppConfig.memberService
call AppConfig.memberRepository
call AppConfig.orderService
```
#### @Configuration과 바이트 코드 조작
- 스프링 컨테이너는 싱글톤 레지스트리이다. 따라서 스프링 빈이 싱글톤이 되도록 보장해주어야 한다.
- 위에서 볼 수 있듯이, 싱글톤 컨테이너는 **@Configuration**을 **AppConfig**에 의해 싱글톤이 보장되도록 해준다.
```
@Test
void configurationDeep() {
    ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
    AppConfig bean = ac.getBean(AppConfig.class);

    System.out.println("bean = " + bean.getClass());
    // 출력 : bean = class hello.core.AppConfig$$EnhancerBySpringCGLIB$$59c4e2c6
}
```
- 위 코드를 보면 알 수 있듯이 **AnnotationConfigApplicationContext**에 파라미터로 넘긴 **AppConfig** 또한 스프링 빈으로 등록된다.
- 그런데, 해당 클래스를 출력해보면 xxxCGLIB가 붙으면서 상당히 복잡해진 것을 볼 수 있다.
- 이는 내가 만든 AppConfig 클래스가 아니라 스프링이 CGLIB라는 바이트코드 조작 라이브러리를 사용해서 AppConfig 클래스를 상속 받은 임의의 클래스를 만들고, 해당 클래스를 스프링 빈으로 등록한 것이다.
![img](https://velog.velcdn.com/images%2Fsyleemk%2Fpost%2F034c21a6-ad52-4149-94b0-7bc3d76c7f0c%2Fimage.png)

### CGLIB란?
- Code Generator Library의 약자로 클래스의 바이트 코드를 조작하여 **프록시** 객체를 생성해주는 라이브러리
> 프록시는 JPA의 실제 엔티티를 필요할때만 꺼내쓸 수 있도록 하는 가짜 객체이다. 
![img](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdRrpB1%2FbtrhVYY0LSG%2FL4NCYfk46v7zpiVkrokPfk%2Fimg.png)
**출처: https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdRrpB1%2FbtrhVYY0LSG%2FL4NCYfk46v7zpiVkrokPfk%2Fimg.png**
- CGLIB를 사용하면 인터페이스가 없어도 구체화 클래스만으로 동적 프록시를 만들 수 있다.
- 외부 라이브러리이지만 스프링 프레임워크 내부에 포함되어 있어, 별도로 라이브러리를 추가하지 않아도 된다.
**출처: https://gymdev.tistory.com/67** 


```
// AppConfig@CGLIB 예상 코드

@Bean
public MemberRepository memberRepository() {

 if (memoryMemberRepository가 이미 스프링 컨테이너에 등록되어 있으면?) {
 return 스프링 컨테이너에서 찾아서 반환;
 } else { //스프링 컨테이너에 없으면
 기존 로직을 호출해서 MemoryMemberRepository를 생성하고 스프링 컨테이너에 등록
 return 반환
 }
```
- @Bean이 붙은 메서드마다 이미 스프링 빈이 존재하면 존재하는 빈을 반환하고, 스프링 빈이 없으면 생성해서 스프링 빈으로 등록하고 반환하는 코드가 동적으로 만들어진다.

>참고 AppConfig@CGLIB는 AppConfig의 자식 타입이므로, AppConfig 타입으로 조회 할 수 있다

### 정리
- 싱글톤 패턴은 클래스 인스턴스를 1개만 생성되도록 보장하는 디자인 패턴이다.
- 그러나 싱글톤은 문제점이 있다. (복잡한 코드, DIP 위반, 구체 클래스 의존 등등)
- 스프링의 싱글톤 컨테이너는 이러한 싱글톤 패턴의 단점을 해결하면서 객체를 싱글톤으로 유지해준다.
- 주의할 점은 싱글톤 방식은 여러 클라이언트가 하나의 같은 객체 인스턴스를 공유하기 때문에 무상태로 설계해야 한다.
- 스프링은 CGLIB라는 바이트코드 조작 라이브러리를 사용해서 AppConfig 클래스를 상속받은 임의의 다른 클래스를 만들고, 그 다른 클래스를 스프링 빈으로 등록한다. 
- @Bean만 사용해도 스프링 빈으로 등록되지만, 싱글톤이 보장되지는 않는다. (여러번 호출됨)


**출처: 김영한 님의 Inflearn '스프링 핵심 원리 - 기본편'**  