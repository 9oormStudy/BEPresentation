# 객체지향 원칙 (SOLID 원칙 - LSP, ISP, DIP)

# LSP(Liskov Substitution Principle)란

리스코프 치환 원칙(LSP)은 바바라 리스코프(Barbara Liskov)가 올바른 상속 관계의 특징을 정의하기 위해 발표한 것으로,`부모 객체를 호출하는 동작에서 자식 객체가 부모 객체를 완전히 대체할 수 있다.`는 객체지향 원칙이다.
다른 관점에서 설명하면 `부모 클래스의 객체는 애플리케이션을 손상시키지 않고, 그 자식 클래스의 객체로 대체할 수 있어야 한다.`는 것인데 이를 위해서 `자식 클래스의 객체가 부모 클래스의 객체와 동일한 방식으로 동작해야 한다`는 것이다.

즉, 리스코프 치환 원칙은 **상속이 올바르게 사용되도록 보장하는 방법**이다.

더 쉽게는 `다형성 원리`를 생각하면 된다.
다형성 기능을 사용하기 위해서는 클래스를 상속시켜 타입을 통합할 수 있게 설정하고, 업캐스팅을 해도 메소드 동작에 문제없게 설계해야 한다.

## LSP 적용 예제

Java에서 LSP가 적용된 예제는 Collection Framework이다.
ArrayList와 LinkedList는 각각 List 인터페이스를 구현한 클래스이다. 그리고 printList 메소드에서는 List 인터페이스를 구현한 어떤 클래스든지 받아들일 수 있다.

<details>
<summary>Collection Framework</summary>
<div markdown="1">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Ft1.daumcdn.net%2Fcfile%2Ftistory%2F99B88F3E5AC70FB419">
</div>
</details>

```java
void LSP() {
    List<String> arrayList = new ArrayList<>();
    arrayList.add("Java");
    arrayList.add("is");
    arrayList.add("fun");

    List<String> linkedList = new LinkedList<>();
    linkedList.add("Java");
    linkedList.add("is");
    linkedList.add("fun");

    printList(arrayList);
    printList(linkedList);
}

void printList(List<String> list) {
    for (String item : list) {
        System.out.print(item + " ");
    }
    System.out.println();
}
```

실생활 예시로는 자동차와 전기자동차를 생각할 수 있다.
자동차를 상위 클래스, 전기자동차를 하위 클래스로 볼 때, 전기자동차는 자동차의 모든 기능을 수행할 수 있어야 한다. 예를 들어, 자동차가 가지고 있는 '운전'과 '정지' 기능은 전기자동차도 동일하게 수행할 수 있어야 한다.

**쉽게 LSP는 `다형성을 지원하기 위한 원칙이다.` 라고 할 수 있다.**

## LSP 위반

자동차와 전기자동차의 관계에 대해 코드 예시이다.

```java
class Car {
    void drive() {
        System.out.println("운전 시작");
    }

    void stop() {
        System.out.println("정지");
    }

    void refuel() {
        System.out.println("주유");
    }
}

class ElectricCar extends Car {
    @Override
    void refuel() {
        throw new UnsupportedOperationException("전기자동차는 주유할 수 없습니다.");
    }
}
```

## 해결

공통 기능만을 포함하는 상위 클래스를 정의하고, 필요한 기능은 하위 클래스에서 구현한다.

```java
class Car {
    void drive() {
        System.out.println("운전 시작");
    }

    void stop() {
        System.out.println("정지");
    }
}

class ElectricCar extends Car {
    void recharge() {
        System.out.println("충전");
    }
}

class GasolineCar extends Car {
    void refuel() {
        System.out.println("주유");
    }
}
```

## LSP 적용 시 고려사항

- 인터페이스 일관성: 하위 클래스는 상위 클래스의 인터페이스와 행동을 유지해야 한다. 이는 메소드 시그니처뿐만 아니라, 그 행동의 의미도 포함한다.

- 사전 조건의 확장 금지: 하위 클래스에서 메소드를 구현할 때, 상위 클래스의 메소드에 정의된 사전 조건을 강화해서는 안 된다.

- 사후 조건의 약화 금지: 하위 클래스는 상위 클래스의 메소드가 보장하는 사후 조건을 준수하거나 강화해야 한다. 사후 조건을 약화시키지 않아야 한다.

- 예외 처리의 일관성: 하위 클래스에서 상위 클래스의 메소드를 오버라이드할 때, 새로운 예외를 던지지 않거나, 상위 클래스의 메소드에서 예상되는 예외 유형만을 던져야 한다.

## LSP 적용의 이점

- 코드 재사용성 향상: LSP를 준수하면, 상위 클래스의 참조를 통해 하위 클래스의 객체를 자유롭게 사용할 수 있어 코드 재사용성이 향상된다.

- 유지보수성 증가: 시스템의 다른 부분을 변경하지 않고도 하위 클래스를 변경하거나 확장할 수 있어 유지보수성이 증가한다.

- 확장성 개선: LSP를 준수하는 설계는 새로운 하위 클래스를 쉽게 추가할 수 있게 해, 시스템의 확장성을 개선한다.

# ISP(Interface Segregation Principle)란

직역하자면 `인터페이스를 분리하는 원칙`이라는 것인데, 풀어 설명하면 `인터페이스를 사용에 맞게 끔 각기 분리해야한다는 설계 원칙`이라고 보면 된다. 또한 클라이언트는 자신이 사용하지 않는 메서드에 의존하면 안 된다는 원칙이라고 할 수 있다.

만약 인터페이스의 추상 메서드들을 범용적으로 이것저것 구현한다면, 그 인터페이스를 상속받은 클래스는 자신이 사용하지 않는 인터페이스마저 억지로 구현 해야 하는 상황이 올 수도 있다.

또한 사용하지도 않는 인터페이스의 추상 메소드가 변경된다면 클래스에서도 수정이 필요하게 된다.

즉, 인터페이스 분리 원칙이란 인터페이스를 잘게 분리함으로써, 클라이언트의 목적과 용도에 적합한 인터페이스 만을 제공하는 것이다.

## ISP 적용 예제

실생활 적용 예제로는 다기능 프린터를 예를 들 수 있다.
복사, 스캔, 팩스 기능을 모두 가진 다기능 프린터가 있다고 하면, 사용자가 복사 기능만 필요한 경우에도 스캔과 팩스 기능까지 모두 포함된 프린터를 사용해야 한다. ISP를 적용하면, 각 기능별로 인터페이스를 분리하여 필요한 기능만을 제공받을 수 있다.

## ISP 위반

SimplePrinter는 복사 기능만 필요하지만, 스캔과 팩스 기능까지 구현해야 한다. 즉, ISP를 위반한다.

```java
interface MultiFunctionPrinter {
    void print();
    void scan();
    void fax();
}

class SimplePrinter implements MultiFunctionPrinter {
    public void print() {
        // 구현
    }

    public void scan() {
        // 필요 없는 기능
        throw new UnsupportedOperationException("Scan 기능은 지원하지 않습니다.");
    }

    public void fax() {
        // 필요 없는 기능
        throw new UnsupportedOperationException("Fax 기능은 지원하지 않습니다.");
    }
}
```

## 해결

ISP를 적용하여 각 기능별로 인터페이스를 분리함으로써, SimplePrinter는 복사 기능만 구현하면 되고, 다기능이 필요한 경우에는 MultiFunctionMachine을 사용하여 필요한 인터페이스를 모두 구현할 수 있다.

```java
interface Printer {
    void print();
}

interface Scanner {
    void scan();
}

interface Fax {
    void fax();
}

class SimplePrinter implements Printer {
    public void print() {
        // 구현
    }
}

class MultiFunctionMachine implements Printer, Scanner, Fax {
    public void print() {
        // 구현
    }

    public void scan() {
        // 구현
    }

    public void fax() {
        // 구현
    }
}
```

## ISP 적용 시 고려사항

- 인터페이스의 크기와 범위: 인터페이스는 가능한 한 작고 특정한 기능에 초점을 맞춰야 한다. 이는 클래스가 자신과 관련 없는 메소드를 구현하는 것을 방지한다.
- 클라이언트의 필요성: 인터페이스는 클라이언트가 실제로 필요로 하는 기능만을 제공해야 한다. 클라이언트의 요구사항을 충분히 이해하고 이를 반영하여 인터페이스를 설계해야 한다.
- 모듈성과 재사용성: 인터페이스 분리는 코드의 모듈성과 재사용성을 향상시킨다. 각 인터페이스가 특정 기능에 집중함으로써, 필요한 기능만을 선택하여 사용할 수 있게 된다.
- 유지보수성: 작고 명확한 인터페이스는 유지보수를 용이하게 한다. 인터페이스가 너무 크거나 복잡하면, 변경 사항이 발생했을 때 이를 반영하기 어려울 수 있다.

## ISP 적용의 이점

- 의존성 감소: ISP를 적용하면 클래스 간의 의존성이 줄어들어, 시스템의 결합도가 낮아진다. 이는 각 컴포넌트를 독립적으로 개발하고 테스트할 수 있게 해준다.
- 코드의 견고성과 유지보수성 향상: ISP를 통해 고결합도를 피하고, 코드의 이해도와 견고성을 높일 수 있다. 이는 장기적으로 유지보수성을 크게 향상시킨다.
- 인터페이스의 명확성: ISP는 인터페이스를 더 세분화하고 특정 기능에 초점을 맞추도록 한다. 이는 라이브러리 사용자가 필요한 기능만을 구현하도록 도와준다.
- 모듈성과 재사용성 향상: ISP를 적용하면 시스템의 모듈성과 컴포넌트의 재사용성이 향상된다. 이는 대규모 시스템 개발에 있어 큰 이점을 제공한다.

# DIP(Dependency Inversion Principle)란

DIP 원칙이란 `고수준 모듈은 저수준 모듈에 의존해서는 안 되며, 둘 다 추상화에 의존해야 한다는 원칙`입니다. 즉, 추상화는 세부 사항에 의존해서는 안 되며, 세부 사항이 추상화에 의존해야 합니다.

쉽게 설명하자면 `객체에서 어떤 Class를 참조해서 사용해야하는 상황이 생긴다면, 그 Class를 직접 참조하는 것이 아니라 그 대상의 상위 요소인 추상 클래스나 인터페이스로 참조하라는 원칙`이다.

객체들이 서로 정보를 주고 받을 때는 의존 관계가 형성되는데, 이 때 객체들은 `나름대로의 원칙`을 갖고 정보를 주고 받아야 하는 약속이 있다. 여기서 `나름대로의 원칙`이란 `추상성이 낮은 클래스보다 추상성이 높은 클래스와 통신을 한다는 것`을 의미하는데 이것이 DIP 원칙이라고 한다.

## DIP 적용 예제

실생활 예제로는 운전자와 자동차의 관계를 생각해 볼 수 있다. 운전자(고수준 모듈)는 특정 자동차 모델(저수준 모듈)에 의존해서는 안 된다. 대신, 운전자는 자동차라는 개념(추상화)에 의존해야 한다. 그렇게 되면 운전자는 어떤 자동차 모델이든 운전할 수 있게 된다.

## DIP 위반

Driver 클래스는 KiaCar 클래스에 직접 의존하고 있어 DIP 원칙을 위반하고 있다.

```java
class Driver {
    private KiaCar car;

    public Driver() {
        this.car = new KiaCar(); // Driver 클래스가 KiaCar 클래스에 직접 의존
    }

    public void drive() {
        car.accelerate();
    }
}

class KiaCar {
    public void accelerate() {
        System.out.println("기아 자동차가 가속합니다.");
    }
}
```

## 해결

```java
interface Car {
    void accelerate();
}

class Driver {
    private Car car;

    public Driver(Car car) {
        this.car = car; // Driver 클래스가 Car 인터페이스에 의존
    }

    public void drive() {
        car.accelerate();
    }
}

class KiaCar implements Car {
    @Override
    public void accelerate() {
        System.out.println("기아 자동차가 가속합니다.");
    }
}

class HyundaiCar implements Car {
    @Override
    public void accelerate() {
        System.out.println("현대 자동차가 가속합니다.");
    }
}
```

## DIP 적용 시 고려사항

- 추상화에 의존: 고수준 모듈과 저수준 모듈 모두 추상화에 의존해야 한다. 이는 구체적인 클래스보다는 인터페이스나 추상 클래스에 의존하는 것을 의미한다.

- 모듈 간 결합도 감소: DIP를 적용함으로써 모듈 간의 결합도를 줄이고, 유연성과 확장성을 높일 수 있다.

- 의존성 주입 사용: 의존성 주입(Dependency Injection)은 DIP를 구현하는 데 흔히 사용되는 기법이다. 생성자, 메소드, 또는 필드를 통해 의존성을 주입할 수 있다.

## DIP 적용의 이점

- 결합도 감소: DIP는 고수준 모듈과 저수준 모듈 간의 결합도를 줄여준다. 이로 인해 시스템의 유지보수성과 확장성이 향상된다.

- 유연성 향상: 추상화에 의존함으로써, 구체적인 구현을 변경하거나 대체하기가 더 쉬워진다. 즉, 시스템의 유연성을 크게 향상시킨다.

- 테스트 용이성: 결합도가 낮은 코드는 테스트하기가 더 쉽다. 의존성을 명시적으로 정의하고 주입할 수 있기 때문에, 단위 테스트 시 모의 객체(Mock Object)를 사용하기가 용이하다.
