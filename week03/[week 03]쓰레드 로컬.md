# 쓰레드 로컬

> 해당 쓰레드만 접근할 수 있는 특별한 저장소

쓰레드 로컬은 각 쓰레드마다 별도의 내부 저장소를 제공하고 관리하기 때문에  동시에 같은 인스턴스의 쓰레드 로컬 필드에 접근해도 문제가 발생하지 않는다.

**쓰레드 로컬은 동시성 문제를 해결할 수 있다.**

#### 동시성 문제
> 같은 인스턴스 필드에 두개 이상의 스레드가 접근해서 값을 **변경**할 때, 저장한 값과 조회 결과값이 달라지는 현상을 의미한다. 지역 변수가 아닌 객체가 하나밖에 없는 싱글톤이나 static 전역 변수에서 발생한다.

스프링에서 빈으로 등록된 객체를 여러 쓰레드에서 사용하게되면 이때 사용되는 객체는 싱글톤이므로 여러 쓰레드는 하나의 인스턴스를 사용하게 된다.
 
한 인스턴스의 필드를 조회만 한다면 문제는 발생하지 않고 필드의 값을 변경하게 된다면 동시성 문제가 발생할 확률이 생긴다.

*동시성 문제 관련*

~~~java
private static int sharedCounter = 0;

public static void main(String[] args) throws InterruptedException {
    Thread thread1 = new Thread(() -> {
        for (int i = 0; i < 1000; i++) {
            incrementCounter();
        }
    });

    Thread thread2 = new Thread(() -> {
        for (int i = 0; i < 1000; i++) {
            incrementCounter();
        }
    });

    thread1.start();
    thread2.start();

    thread1.join();
    thread2.join();

    System.out.println("기대 값은 2000 출력은? " + sharedCounter);
}

public static void incrementCounter() {
    sharedCounter++;
}
~~~

*실행마다 다른 출력*
~~~groovy
기대 값은 2000 1436
기대 값은 2000 1930
기대 값은 2000 1524
기대 값은 2000 2000
~~~

여러 쓰레드가 동시에 메소드 호출할 때 동시성 문제가 발생할 수 있음

### 쓰레드 로컬 동작 방식

해시맵 방식으로 동작한다.
ThreadLocal 의 내부 클래스인 ThreadLocalMap 객체를 참조 필드로 가지고 있기 때문에 해당 필드에 새로 생성한 맵을 저장해준다.

*ThreadLocalMap 생성자*
~~~java
ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
    table = new Entry[INITIAL_CAPACITY];
    int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
    table[i] = new Entry(firstKey, firstValue);
    size = 1;
    setThreshold(INITIAL_CAPACITY);
}
~~~


*쓰레드 로컬 사용 예시*
~~~java
private static ThreadLocal<Integer> threadLocalCounter = ThreadLocal.withInitial(() -> 0);

    public static void main(String[] args) throws InterruptedException {
        Thread thread1 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                incrementCounter();
            }
            System.out.println("쓰레드 1 Counter: " + threadLocalCounter.get());
            threadLocalCounter.remove(); // 스레드 로컬 데이터 정리
        });

        Thread thread2 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                incrementCounter();
            }
            System.out.println("쓰레드 2 Counter: " + threadLocalCounter.get());
            threadLocalCounter.remove(); // 스레드 로컬 데이터 정리
        });

        thread1.start();
        thread2.start();

        thread1.join();
        thread2.join();
    }

    public static void incrementCounter() {
        int current = threadLocalCounter.get();
        threadLocalCounter.set(current + 1);
    }
~~~

~~~groovy
쓰레드 1 Counter: 1000
쓰레드 2 Counter: 1000
~~~

#### 쓰레드 로컬 주의사항

*주의사항 사용 예시*
~~~java
public class UserService {
    private static final ThreadLocal<User> currentUser = new ThreadLocal<>();

    public void login(User user) {
        currentUser.set(user);
    }

    public void logout() {
        currentUser.remove();
    }
}
~~~

쓰레드 로컬을 모두 사용하고 나면 remove()를 호출해 쓰레드 로컬에 저장된 값을 제거해줘야 한다. was횐경에서 쓰레드를 사용 후 제거하지않고 쓰레드 풀에 반환해 재사용하기 때문에 제거하지 않으면 이후 요청에 같은 쓰레드를 사용할 경우 기존에 남아있던 데이터가 다른 유저에게 노출이 될 경우가 발생할 수 있다.

### 스프링과 쓰레드 로컬

*스프링과 쓰레드 로컬 사용 예시*
~~~java
@Component
public class UserContext {
    private static final ThreadLocal<User> currentUser = new ThreadLocal<>();

    public static void setCurrentUser(User user) {
        currentUser.set(user);
    }

    public static User getCurrentUser() {
        return currentUser.get();
    }

    public static void clear() {
        currentUser.remove();
    }
}
~~~

~~~java
@RestController
public class UserController {

    @GetMapping("/user/profile")
    public String userProfile() {
        User user = UserContext.getCurrentUser();
        if (user != null) {
            return user.getName() + " 의 프로필" ;
        }
        return "로그인하세요";
    }

    @GetMapping("/login")
    public String login() {
        // 임시 사용자 생성
        User user = new User("MJ");
        UserContext.setCurrentUser(user);
        return "로그인 성공";
    }

    @GetMapping("/logout")
    public String logout() {
        UserContext.clear();
        return "로그아웃";
    }
}
~~~

* UserContext **클래스**: 사용자 정보를 ThreadLocal에 저장하고 관리 
  이 클래스는 스레드 안전하며 각 요청은 자신의 사용자 정보에 독립적으로 접근할 수 있다.
* **컨트롤러 메소드**: 사용자 로그인, 로그아웃, 프로필 정보 조회 등을 처리
  이를 통해 각 요청은 사용자 컨텍스트에 저장된 정보에 접근할 수 있다.

## 정리

#### 동시성 문제
동시성 문제는 여러 스레드가 동일한 데이터에 동시에 접근하고 수정할 때 발생. 
특히, 싱글톤 빈이나 static 변수와 같이 공유되는 자원에 대한 동시 수정이 이루어질 때, 
데이터의 불일치와 예측 불가능한 결과가 발생할 수 있다. 
이 문제는 데이터 손실과 애플리케이션의 신뢰성 저하를 초래할 수 있습니다.

#### 쓰레드 로컬
ThreadLocal은 각 스레드에 데이터의 사본을 제공해, 스레드가 데이터를 독립적으로 수정할 수 있게 해준다. 
이는 동시성 문제를 효과적으로 해결하는 방법 중 하나로, 스레드 간 데이터 충돌 없이 각 스레드가 자신만의 고유한 저장소를 관리할 수 있게 한다.

#### 쓰레드 로컬 주의사항
ThreadLocal을 사용할 때는 주의해야 할 점이 있다. 각 스레드의 작업이 완료된 후 remove() 메소드를 호출하여 ThreadLocal 변수에서 데이터를 제거하는 것이다. 메모리 누수를 방지하고, 스레드 풀 환경에서 스레드가 재사용될 때 이전의 데이터가 남아 있지 않도록 보장한다.
