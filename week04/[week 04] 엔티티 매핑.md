# 엔티티 매핑

> 자바 클래스와 데이터베이스 테이블 간의 링크를 정의하는 작업
> **자바 언어로 정의된 엔티티 클래스와 데이터베이스의 릴레이션 간에 어떤 관계를 맺을지 지정하는 것**

JPA는 이를 통해 데이터베이스의 레코드와 자바 객체 간의 자동 영속화를 제공
*객체 지향 프로그래밍과 관계형 데이터베이스 간의 패러다임 불일치 해결 방법 제시*

~@Entity~ 에노테이션을 클래스에 붙이면 해당 클래스는 JPA 엔티티임을 나타낸다.
JPA는 이 에노테이션이 붙은 클래스를 데이터베이스의 테이블과 매핑해 객체를 영구 저장소에 저장하고 관리한다.

### @Entity 특징
1. JPA에서 관리하는 엔티티 클래스는 반드시 에노테이션이 붙어야 한다.
2. 클래스는 기본 생성자가 반드시 필요하고, 파라미터가 없는 기본 생성자를 갖고 있어야 한다.
3. 엔티티의 상태를 추적하고, 영속성 컨텍스트를 통해 관리하는 등 기능을 제공한다.

## 주요 엔티티 매핑 에노테이션
- @Entity : 해당 클래스가 JPA에 관리하는 엔티티임을 지정
- - name : 엔티티 이름 지정 (기본적으로 클래스 이름과 동일)
  - catalog : 엔티티 카탈로그 지정 (엔티티가 속한 카탈로그 지정할 때 사용)
  - schema : 엔티티 스키마 지정 (집합을 그룹화하는 개념으로 사용)
  - indexes : 엔티티 인덱스를 정의
  - unipueConstraints : 유니크 제약 조건을 정의
  - listeners : 이벤트를 처리하는 리스너 지정
- @Table : 엔티티와 매핑할 테이블을 지정 (name 속성으로 테이블 이름 지정 가능)
- @Id : 엔티티의 기본 키 필드로 지정
- @GeneratedValue : 키 값을 자동으로 생성 (Auto Increment로 지정)
- @Column : 필드를 데이터베이스 칼럼에 매핑 (nullable,length,unique 등 설정 가능)
- @Temporal : 자바의 ~…util.Date~, ~…util.Calendar~ 타입을 매핑
- @Enumrated : Enum 타입을 ORDINAL 또는 STRING 형태로 매핑

### 관계 매핑

#### 1:1 매핑
- 양쪽이 서로 하나의 관계만 가짐
- 외래 키가 어디에 있든 상관 없음

*단방향 관계*
~~~java
@Entity
public class Store {

    @Id
    @GeneratedValue
    private Long id;
    
    private String storeName;
       
}


@Entity
public class Consumer {

    @Id
    @GeneratedValue
    private Long id;
    
    private String consumerName;
    
    @OneToOne
    @JoinColumn(name = "Store_id")
    private Store Store;
    
}

~~~

*양방향 관계*
~~~java
@Entity
public class Store {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String storeName;
    
    @OneToOne(mappedBy = "parking")
    private Charge charge;
    
}


@Entity
public class Consumer {

    @Id
    @GeneratedValue
    private Long id;
    
    private String consumerName;
    
    
    @OneToOne
    @JoinColumn(name = "Store_id") 
    // Store을 연관관계 주인으로 하고 싶다면 mappedBy, @joinColumn의 위치를 서로 바꿔준다.
    private Store Store;
    
}

~~~
#### M : 1 매핑

*단방향 관계*
~~~java
@Entity
public class Store {

    @Id
    @GeneratedValue
    private Long id;
    
    private String sToreName;
    
    @ManyToOne
    @JoinColumn(name="consumer_id")
    private Consumer consumer;
    
}


@Entity
public class Consumer {

    @Id
    @GeneratedValue
    private Long id;
    
    private String consumerName;
    
}

~~~

* @ManyToOne : M:1 관계를 표현하는 어노테이션
  * @ManyToOne이 붙은 엔티티가 M이고 반대 엔티티가 1일 때 붙인다.
* @JoinColumn(name="") : 외래 키를 정의하는 어노테이션


*양방향 관계*
~~~java
@Entity
public class Store {

    @Id
    @GeneratedValue
    private Long id;
    
    private String storeName;
    
    @ManyToOne
    @JoinColumn(name="consumer_id")
    private Consumer consumer;
    
}


@Entity
public class Consumer {

    @Id
    @GeneratedValue
    private Long id;
    
    private String consumerName;
    
    //양방향 매핑을 위한 추가
    @OneToMany(mappedBy = "cunsumer")
    private List<Store> Store = new ArrayList<>();
    
}
~~~

* JPA는 두 개의 연관관계 중 하나를 고르게 하기 위해 mappedBy를 설정
* mappedBy가 없는 엔티티가 연관관계의 주인
  * mappedBy가 없는 Parking 엔티티에 외래 키가 생성

#### 1 : M 매핑

일대다 매핑은 권장되지 않고, 다대일 관계로 맺는 것이 좋다.

#### M : M 매핑

테이블 사이 중간 테이블을 두어 관계를 풀어야하지만 실무에선 한계가 존재하기에 지양한다.

- 중간 테이블을 묵시적으로 생성해 주기 때문에 복잡한 조인 쿼리가 발생
- 중간 테이블에 필요한 추가 칼럼 사용할 수 없다.
- - 추가된 칼럼에 대해 매핑이 되지 않기 때문

## 데이터 베이스 스키마 자동 생성

> DB스키마 자동 생성은 JPA 사용할 때 제공되는 기능 중 하나로, JPA는 객체와 DB 테이블을 매핑하기 위해 사용되는 ORM기술로, 엔티티 클래스를 바탕으로 **DB 테이블을 자동**으로 생성할 수 있다.

*스프링 프레임워크 적용*
~~~properties
 # application.properties -> 스키마 자동 생성
 spring.jpa.hibernate.ddl-auto=create
 # application.properties -> 스키마 자동 생성을 비활성화
 spring.jpa.hibernate.ddl-auto=none
~~~

#### 스키마 자동 생성 주의 점

이미 DB에 데이터가 존재하는 경우, 
**스키마 자동 생성 시 기존 데이터가 삭제될 수 있으므로, 테스트 환경 등 개발 사에만 사용**

*배포나 운영 환경에서는 spring.jpa.hibernate.ddl-auto 값을 none으로 설정하거나 주석 처리하여 스키마 자동 생성을 비활성화하고, 데이터베이스 스키마를 수동으로 관리하는 것을 권장*

#### 스키마 자동 생성 속성
- create : 시작 시 스키마 생성, 이미 존재하면 삭제하고 다시 생성
- update : 변경된 부분만 반영 (운영 DB에선 절대 사용하면 안됨)
- drop : 종료 시 스키마 삭제
- none : 스키마 생성 액션을 수행하지 않고, 개발자가 수동으로 스키마 생성
- validate : Entity와 테이블이 정상 매핑되었는지 확일할 때 사용

*개발 초기엔 create, update 사용*

