**Lombok :** Java 라이브러리중 하나로 반복되는 getter, setter, toString .. 등의 반복 메서드 작성 코드를 줄일수 있는 코드 다이어트 라이브러리이다.

**그래서 롬복이란?**

LomBok이란 **어노테이션 기반으로 코드 자동완성 기능을 제공하는 라이브러리**이다.

Spring, Spring Boot 로 Web 개발을 하다보면 **반복되는 코드**가 자주 등장하며 가독성을 떨어트린다.

예를 들어보면 **Getter, Setter, ToString, Constructor(생성자)**가 대표적인 예제일 것이다.

**ex)**

```java
public class Store extends Common {

    private String companyName;                   
    private String industryTypeCode;              
    private String businessCodeName;                
    private String industryName;                     
    private String telephone;                                   
    private String regionMoneyName;                          
    private boolean isBmoneyPossible;                    
    private boolean isCardPossible;                        
    private boolean isMobilePossible;                       
    private String lotnoAddr;                                 
    private String roadAddr;                                 
    private String zipCode;                                  
    private double longitude;                               
    private double latitude;                                    
    private String sigunCode;                                  
    private String sigunName;   
    
     public String getCompanyName() {
        return companyName;
    }

    public void setCompanyName(String companyName) {
        this.companyName = companyName;
    }

    public String getIndustryTypeCode() {
        return industryTypeCode;
    }

    public void setIndustryTypeCode(String industryTypeCode) {
        this.industryTypeCode = industryTypeCode;
    }
     ....                          //엄청 많은 getter,setter 등장
    
  }                             
```

위 코드는 만약 가게를 만들 때 가게의 정보를 담는 vo이다.

Store에서는 생성자, toString 도 Override 해야하는 등의 추가작업이 이루어지면 위 코드에서만 

해도 엄청나게 많은 Getter,Setter,toString등의 메소드가 된다.

위 코드는 반복적인 긴 코드로 인해 개발자는 스트레스가 쌓이고(?) 가독성이 떨어지는 단점도 발생할 것이다.

위 코드에 Lombok을 적용한다면 아래 코드와 같이 단순화가 가능하다.

```java
@Getter
@Setter
public class Store extends Common {

    private String companyName;                              
    private String industryTypeCode;                          
    private String businessCodeName;                         
    private String industryName;                             
    private String telephone;                                  
    private String regionMoneyName;                            
    private boolean isBmoneyPossible;                          
    private boolean isCardPossible;                           
    private boolean isMobilePossible;                         
    private String lotnoAddr;                                 
    private String roadAddr;                                  
    private String zipCode;                                  
    private double longitude;                                 
    private double latitude;                                 
    private String sigunCode;                                
    private String sigunName;                                 
		
}
```

코드에 상단에 @Getter, @Setter 어노테이션만 추가 하였는데도 코드가 상당히 다이어트화 된 것을 볼 수 있다.

**Lombok 어노테이션 정리**

| @Getter | code가 컴파일 될 때 getter 메서드들을 생성한다. |
| --- | --- |
| @Setter | code가 컴파일 될 때 setter 메서드들을 생성한다. |
| @ToString | toString() 메서드를 생성한다. |
| @EqualsAndHashCode | 사용 객체에 대해서 equals(), hashCode() 메서드를 생성한다. |
| @Data | @Getter(모든속성), @Setter(final이 붙지 않은), @ToString, 
@EqualsAndHashCode, @RequiredArgsConstructor
위의 어노테이션들을 합쳐둔 어노테이션이다.  |
| @NoArgsConstructor | 파라미터(매개변수)가 없는 생성자를 생성한다. |
| @RequiredArgsConstructor | final, @NonNull이 있는 필드를 포함하여 생성자를 생성한다. |
| @AllArgsConstructor | 모든 필드를 파라미터(매개변수)로 갖는 생성자를 생성한다. |
| @Builder | 해당 클래스에 빌더 패턴을 사용할 수 있도록 해준다. |
| @Log | 컴파일시 : private static final java.util.logging.Logger log = java.util.logging.Logger.getLogger(this.class.getName());
-> 해당 코드가 생성되는것이다. |
| @Log4j, @Slf4J | Log4J(Slf4J) 설정을 이용하여 로그 기능 사용할 수있다.마찬가지로 log 변수를 통해 사용 한다. |
| @Synchronized |  동기화 관련 문제 발생을 해당 어노테이션을 통해 가상의 필드 레벨에서 조금이나마 안전하게 락을 걸어준다. |
| @NonNull | 필드의 값이 null이 될 수 없음을 명시해준다. |
| @Value |  모든 필드를 Private, Final 로 설정하고, Setter를 생성하지 않는다.(상수로 만들어준다.)
 FInal 이 붙기 때문에 Setter는 존재할 수가 없는것이다. |

많은 어노테이션이 있지만 안정화 이슈로 자주 사용하는것은 @Getter,@Setter,@ToString,@Data 이다.

**-Lombok 의 장점-**

- 어노테이션 기반의 코드 자동 생성을 이용한 생산성 향상
- 반복되는 코드를 다이어트화 시켜 가독성 및 유지보수성 향상

**-Lombok 의 단점-**

- 개발자가 코드 분석을 할 때 코드 일부가 숨겨져 동작 방식을 파악하기 어려워지고
    
    Lombok을 처음 사용하는 개발자들은 생성된 코드와 어노테이션의 동작을 완전히 이해하지 못하는 경우가 많다.
    
- 가끔 IDE에서 자동 완성과 분석이 제대로 작동하지 않아 개발자들이 Lombok이 생성한 코드를 직접 확인해야 하고 수정할 때가 생긴다.
- Java 와 IDE 의 새 버전이 나왔을 때 Lombok이 정상 작동하지 않을 수 있다. ( 해결 버전을 기다려야하는 경우가 있다.)
- Lombok으로 이루어진 코드의 경우 프로젝트를 공유하는 다른 개발자도 Lombok을 설치하고 사용해야 한다.

개인적인 생각

lombok은 개발자의 스트레스를 줄여(?) 준다는 장점이 있지만 실무에서 간단히 믿고 쓰기에는 아쉬운거 같다.
