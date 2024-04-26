# 스프링에서의 템플릿 콜백 패턴

템플릿 콜백 패턴은 전략 패턴의 변형된 형태이다. 전략 배턴은 변화되는 부분을 매번 인터페이스를 상속하는 클래스로 만들고 외부에서 구현 클래스를 주입한다. 반면 템플릿 콜백 패턴은 변화되는 부분을 매번 독립된 클래스로 만드는 것이 아니라 익명 내부 클래스를 활용하는 방식이다.

### 1. 전략 패턴이란?

---

변하지 않는 부분(context)과 변하는 부분(strategy)을 분리하여 필요에 따라 전략을 동적으로 바꿀 수 있도록 해주는 디자인 패턴이다. 즉, DI 방식을 사용하여 클라이언트가 Context에게 문맥에 맞는 Strategy를 주입하는 패턴을 말한다.

### 1.1. 전략 패턴 예시

학생이 선생님께 공부를 배우고 있다. 이때 학생은 빨간색, 파란색, 검은색 펜을 사용하여 필기를 할 수 있는데 선생님이 펜을 골라주어야 필기를 할 수 있는 상황이다.

```java
// 전략 클래스
public interface Strategy {
    // 구현을 강제할 추상 메소드
    public abstract void ChoosePen();
}
```

```java
// 검은펜 전략
public class BlackPen implements Strategy {
    public void ChoosePen() {
        System.out.println("검은펜을 잡았습니다.");
    }
}
```

```java

// 파란펜 전략
public class BluePen implements Strategy {
    public void ChoosePen() {
        System.out.println("파란펜을 잡았습니다.");
    }
}
```

```java

// 빨간펜 전략
public class RedPen implements Strategy{
    public void ChoosePen() {
        System.out.println("빨간펜을 잡았습니다.");
    }
}
```

```java
// Client가 준 전략을 수행하는 Context 클래스
public class Student {
    public void takeNotes(Strategy strategy) {
        System.out.println("=== 선생님이 펜을 주십니다. ===");
        strategy.ChoosePen();
        System.out.println("필기를 시작합니다.");
    }
}
```

```java
public class Teacher {
    public static void main(String[] args) {
        // Context 객체 생성
        Student student = new Student();

        // Strategy 객체 생성
        BlackPen black = new BlackPen();
        RedPen red = new RedPen();
        BluePen blue = new BluePen();

        // Context에 Strategy '주입'
        student.takeNotes(black);
        student.takeNotes(red);
        student.takeNotes(blue);
    }
}
```

### 2. 템플릿 콜팩 패턴

---

템플릿 콜백 패턴은 전략 패턴의 기본 구조에 인터페이스를 상속하는 클래스를 만들지 않고 익명 내부 클래스를 활용하는 방식이다. 전략 패턴의 context를 템플릿이라 부르고, 익명 내부 클래스로 만들어지는 객체를 콜백이라 한다. 전략 패턴에 정의했던 strategy 구현 클래스를 client나 context에 포함시켜 코드를 간결하게 만든다.

> 내부클래스 : 클래스 안에 생성된 클래스이다.

> 익명클래스 : 이름이 없는 클래스로, 선언과 객체 생성이 동시에 된다.

### 2.1. 템플릿 콜백 패턴 예시

전략 패턴의 학생 예제 코드를 템플릿 콜백 패턴으로 바꾸었다.

**예시 1. 전략 구현 클래스들의 내용을 client에 포함**

```java
// 전략 패턴 (동일)
public interface Strategy {
    public abstract void ChoosePen();
}
```

```java
// Client가 준 전략을 수행하는 Context 클래스 (동일)
public class Student {
    public void takeNotes(Strategy strategy) {
        System.out.println("=== 선생님이 펜을 주십니다. ===");
        strategy.ChoosePen();
        System.out.println("필기를 시작합니다.");
    }
}
```

```java
public class Teacher {
    public static void main(String[] args) {
        // Context 객체 생성
        Student student = new Student();

        // Context에 '익명 내부 클래스'로 Strategy 주입
        student.takeNotes(new Strategy() {
            public void ChoosePen() {
                System.out.println("검은펜을 잡았습니다.");
            }
        });

        student.takeNotes(new Strategy() {
            public void ChoosePen() {
                System.out.println("빨간펜을 잡았습니다.");
            }
        });

        student.takeNotes(new Strategy() {
            public void ChoosePen() {
                System.out.println("파란펜을 잡았습니다.");
            }
        });
    }
}
```

**예시 2. 중복된 구현 내용을 context에 포함시켜 중복 최소화 - 리팩토링**

```java
public interface Strategy {
    public abstract void ChoosePen();
}
```

```java
public class Student {
    public void takeNotes(String Pen) {
        System.out.println("=== 선생님께서 펜을 주십니다. ===");
        takePen(Pen).ChoosePen();
        System.out.println("필기를 시작합니다.");
    }

    // Strategy를 익명 클래스로 반환하는 메서드
    private Strategy takePen(String Pen) {
        // 익명 클래스 사용
        return new Strategy() {
            @Override
            public void ChoosePen() {
                System.out.println(Pen + "을 잡았습니다.");
            }
        };
    }
}
```

```java
public class Teacher {
    public static void main(String[] args) {
        Student student = new Student();

        student.takeNotes("검정펜");
        student.takeNotes("빨간펜");
        student.takeNotes("파란펜");
        student.takeNotes("무지개 형관펜");
    }
}
```

### 2.2. 템플릿 콜백 패턴의 장단점

- 장점
  - 코드의 재사용성과 유연성을 높일 수 있다.
  - 팩토리 객체 없이 해당 객체를 사용하는 메소드에서 인터페이스의 전략을 선택 할 수 있다.
- 단점
  - 인터페이스를 사용하긴 하지만, 실제 사용할 클래스를 직접 선언할 경우 결합도가 증가한다.
