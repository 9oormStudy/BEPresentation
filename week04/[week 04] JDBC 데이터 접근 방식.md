# JDBC 직접 사용

직접 데이터베이스에 접근하고  
직접 sql을 전달하여 사용

```java
public class JdbcExample {

    private DataSource dataSource;

    public JdbcExample(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    public List<String> fetchEmployees() {
        List<String> employees = new ArrayList<>();
        String sql = "SELECT name, department FROM employees";

        try (Connection connection = dataSource.getConnection();
             PreparedStatement pstmt = connection.prepareStatement(sql);
             ResultSet resultSet = pstmt.executeQuery()) {

            while (resultSet.next()) {
                String employeeInfo = resultSet.getString("name") + " - " + resultSet.getString("department");
                employees.add(employeeInfo);
            }
        } catch (Exception e) {
            System.err.println("Error fetching employees: " + e.getMessage());
        }
        return employees;
    }
}
```

.

# JdbcTemplate
JDBC를 사용할 때 반복되는 부분들을 효과적으로 처리하기 위한 템플릿

```java
public class JdbcExample {

    private JdbcTemplate jdbcTemplate;

    public JdbcExample(DataSource dataSource) {
        this.jdbcTemplate = new JdbcTemplate(dataSource);
    }

    public List<String> fetchEmployees() {
        String sql = "SELECT name, department FROM employees";
        return jdbcTemplate.query(sql, (rs, rowNum) -> 
            rs.getString("name") + " - " + rs.getString("department")
        );
    }
}
```

### 장점

- **반복 문제 해결**  
JDBC를 직접 사용할 때 발생하는 반복 작업을 처리해준다.  
개발자는 SQL 작성, 파라미터 정의, 응답 값 매핑만 해주면 된다.

- **편리한 설정**  
`spring-jdbc` 라이브러리에 포함되어 있기 때문에,  
스프링으로 JDBC를 사용할 시 기본적으로 사용되며 복잡한 설정 없이 바로 사용할 수 있다.

### 단점
- **불편한 동적 쿼리**  
```java
String sql = "select id, item_name, price, quantity from item";

if (StringUtils.hasText(itemName) || maxPrice != null) {
	sql += " where";
}

boolean andFlag = false;
if (StringUtils.hasText(itemName)) {
	sql += " item_name like concat('%',:itemName,'%')";
    andFlag = true;
}

if (maxPrice != null) {
	if (andFlag) {
    	sql += " and";
    }
    sql += " price <= :maxPrice";
}

log.info("sql={}", sql);
return template.query(sql, param, itemRowMapper());
```

.

# SQL Mapper

`JdbcTemplate`보다 많은 기능을 제공하는 `SQL Mapper`  
ex. `MyBatis`

```java
public class MybatisExample {

    private SqlSessionFactory sqlSessionFactory;

    public MybatisExample(SqlSessionFactory sqlSessionFactory) {
        this.sqlSessionFactory = sqlSessionFactory;
    }

    public List<Employee> fetchEmployees() {
        try (SqlSession session = sqlSessionFactory.openSession()) {
            EmployeeMapper mapper = session.getMapper(EmployeeMapper.class);
            return mapper.findAllEmployees();
        }
    }
}
```

### 장점
- **더 나은 동적 쿼리**  
SQL을 XML에 짧고 편리하게 작성할 수 있다.
```xml
<select id="findAll" resultType="Item">
  select id, item_name, price, quantity
  from item
  <where>
    <if test="itemName != null and itemName != ''">
      and item_name like concat('%',#{itemName},'%')
    </if>
    <if test="maxPrice != null">
      and price &lt;= #{maxPrice}
    </if>
  </where>
</select>
```

### 단점
- **유지보수의 어려움**  
쿼리가 숨겨지게 되어 전체 흐름을 파악하기 어려워진다.  
별도의 설정 파일 및 XML 파일이 많아지면 관리가 어렵다.  
성능 최적화에 제한적이다.  

.

# JPA
자바 표준 **ORM** 데이터 접근 기술

```java
@Service
public class JpaExampleService {

    private final EmployeeRepository employeeRepository;

    @Autowired
    public JpaExampleService(EmployeeRepository employeeRepository) {
        this.employeeRepository = employeeRepository;
    }

    public List<Employee> fetchEmployees() {
        return employeeRepository.findAll();
    }
}
```

**ORM** : 객체 지향 프로그래밍 언어를 사용하여 호환되지 않는 유형의 데이터를 변환하는 기법  
복잡한 SQL 쿼리 없이도 데이터베이스 데이터를 객체 형태로 쉽게 다룰 수 있게 해준다.

```java
@Data		// lombok
@Entity		// JPA 사용 객체 명시
public class Item {
	
    @Id 	// 테이블의 PK 매핑
    @GeneratedValue(strategy = GenerationType.IDENTITY)		// PK 생성 방식 명시
    private Long id;
    
    @Column(name = "item_name", length = 10)	// Hibernate의 경우 카멜케이스를 언더스코어로 자동변환
    private String itemName;
    private Integer price;						// @Column 생략 시 필드명으로 매핑
    private Integer quantity;
    
    public Item() {								// JPA는 기본 생성자 필수
    }
    
    public Item(String itemName, Integer price, Integer quantity) {
    	this.itemName = itemName;
    	this.price = price;
    	this.quantity = quantity;        
    }
    
}
```

실무에서는 **스프링 데이터 JPA**와 **QueryDSL**을 함께 사용  

### 장점
- **객체 중심 개발**  
  DB 작업을 객체 지향적으로 개발할 수 있다.  
- **유지보수**  
  엔티티 모델을 중심으로 코딩하기에, 코드의 명확성과 유지보수성이 향상  
  DB 구조 변경 시, 엔티티 클래스만 업데이트하면 된다.  

### 단점

- **어려움**  
- **오버헤드**  
  JPA 구현체는 실행 시 많은 자동 처리 작업을 수행,  
  간단한 쿼리에서도 불필요한 오버헤드가 발생 가능

