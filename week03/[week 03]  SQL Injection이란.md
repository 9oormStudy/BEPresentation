# SQL Injection 이란?

## SQL Injection 이란 무엇인가?

> Injection : `주입`

웹 애플리케이션이 백엔드에서 구동 중인 데이터베이스에 질의를 하는 과정에 사용되는 SQL 쿼리를 의도적으로 조작해 데이터베이스를 대상으로 공격자가 의도한 악의적인 행위를 할 수 있게 하는 코드 인젝션 공격 방법이다.

인젝션 공격은 OWASP Top10 중 첫 번째에 속해 있으며, 공격이 비교적 쉬운 편이고 공격에 성공할 경우 큰 피해를 입힐 수 있는 공격이다.

## SQL Injection 으로 인한 피해 사례 참고

![img](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Ft1.daumcdn.net%2Fcfile%2Ftistory%2F99D4993C5C8890FB1D)

**2017년 3월에 일어난 “여기어때” 의 대규모 개인정보 유출 사건**

고객에게 불쾌한 내용을 담은 문자메시지가 고객 4천여명에게 문자가 발송되면서 문제가 떠올랐다.

해킹으로 이용자명, 휴대폰 번호, 숙박 이용 정보 323만 건 이 유출됐다.
해커는 다양한 공격 시도를 했고, 그 중 SQL 주입 공격이 발견되었다.

## SQL Injection 동작 원리

### 로그인 요청 과정

![정상적인 로그인 요청 수행 과정](https://www.bugbountyclub.com/uploads/images/training/1644468226.png)
먼저 정상적인 로그인 요청 수행 과정을 사진을 통해 볼 수 있다.

- 사용자는 웹애플리케이션에서 데이터베이스와 연동된 기능을 사용할 때 사용자에 의해 입력된 데이터(or 웹애플리케이션에 의해 셋팅된 데이터)는 HTTP 요청을 통해 웹 애플리케이션으로 전송된다. (GET 매개변수 or POST Body 매개변수 등)
- 전송된 데이터는 서버측 스크립트에 의해 미리 정의 및 저장되어있던 SQL 쿼리 안의 지정된 위치로 대입된다.
- 서버측 스크립트는 데이터베이스에 완성된 SQL 쿼리 실행을 요청한다.
- 데이터베이스는 SQL 쿼리를 실행한 후 조건에 부합하는 레코드셋이나 Boolean 값(참/거짓) 또는 오류를 반환해준다.
- 서버측 스크립트는 데이터베이스에서 반환된 데이터에 맞게 HTTP 응답 메시지를 만들어 사용자에게 회신한다.

![SQL Injection을 통한 로그인 우회](https://www.bugbountyclub.com/uploads/images/training/1644468194.png)
취약하게 구현된 로그인 기능은 SQL Injection을 통해 인증을 우회하고 다른 사용자의 계정으로 로그인 할 수 있다.

#### 공격자

- 아이디 입력 : `admin' or '1'='1`
- 비밀번호 입력 : 원하는 임의의 문자열

공격자가 입력한 아이디와 비밀번호 값은 HTTP 요청을 통해 서버로 전달되어 아래와 같은 쿼리를 만들게 된다.

```SQL
SELECT * FROM users WHERE id='admin' or '1'='1' AND password='anything';
```

그럼 위 SQL 쿼리는 웹 애플리케이션에 admin이라는 사용자 계정이 존재한다면 패스워드가 일치하는지 여부와는 상관없이 admin 계정으로 로그인하게 된다.

여부가 상관없이 로그인이 되는 이유는 바로 SQL 쿼리의 논리 연산을 조작하여 무조건 참(TRUE)이 되었기 때문이다.

그럼 어떻게 TRUE가 될 수 있었을까?

![sql문](https://www.bugbountyclub.com/uploads/images/training/1616992423.png)

| Condition  | 내용                | 조건식의 T/F 여부               |
| ---------- | ------------------- | ------------------------------- |
| Condition1 | id='admin'          | TRUE (admin 계정이 있다고 가정) |
| Condition2 | '1'='1'             | TRUE                            |
| Condition3 | password='anything' | FALSE                           |

AND `연산자의 우선순위`가 OR 연산자의 우선순위보다 높다는 것을 생각해야한다.

![](https://www.bugbountyclub.com/uploads/images/training/1616991390.png)

그럼 Condition2와 Condition3으로 FALSE이 되고, 그 FALSE 값과 Condition1이 만나 TRUE가 되어 최종적으로는 admin 계정으로 로그인에 성공하게 되는 것이다.

### 게시글 조회 과정

![정상적인 게시글 조회 요청 수행 과정](https://www.bugbountyclub.com/uploads/images/training/1644469018.png)

게시판 기능에서 사용자가 4215번의 특정 게시글을 클릭하면 해당 게시글의 내용을 보여주기 위해 아래의 SQL 쿼리가 사용된다.

```sql
SELECT brd_no, subject, content, author, post_date FROM board WHERE brd_no=4215;
```

#### 공격자

- SQL 쿼리의 종료를 알리는 세미콜론(;)을 사용한다.
- 본인이 의도한 악의적인 SQL 문장을 이용하여 "brd_no"라는 GET 매개변수를 조작한다.
- 9999는 존재하지 않는 게시글의 번호로 가정한다.

```sql
/board/view.php?brd_no=9999;Drop%20table%20users;
```

위 요청을 통해 전달된 "brd_no" 매개변수의 값은 게시글을 조회하기 위해 미리 정의되어있던 SQL 쿼리에 포함되고 SQL 쿼리는 데이터베이스에서 실행된다.
하지만 실행되는 SQL 쿼리는 한 개가 아니다. 의도적으로 추가한 세미콜론(;)에 의해 SQL 쿼리는 두 개의 쿼리로 나뉘어졌고 원래 SQL 쿼리와 공격자가 추가한 악의적인 SQL 쿼리가 모두 하나의 트랜잭션에서 실행되게 된다.

```sql
SELECT brd_no, subject, content, author, post_date FROM board WHERE brd_no=9999;Drop table users;
```

만약 웹 애플리케이션이 사용하는 데이터베이스 계정이 테이블 삭제 권한을 보유하고 있다면 `Drop table users;` 명령에 의해 users 테이블은 삭제되고, 서비스에 큰 손상을 주게 된다.

![SQL Injection을 통한 악의적인 명령 실행](https://www.bugbountyclub.com/uploads/images/training/1644469586.png)

## SQL Injection 공격 종류 및 방법

- **Error based SQL Injection**

  SQL 쿼리에서 의도적으로 오류를 발생시켜 정보를 추출하는 기법 (위 로그인 요청 과정 예시 참고)

- **UNION based SQL Injection**

  UNION SQL 명령어를 사용하여 두 개 이상의 SELECT 문을 결합하여 추가적인 정보를 얻는 방법

- **Blind SQL Injection**

  직접적인 데이터베이스 오류 메시지를 통해 정보를 얻지 못하는 상황에서 참/거짓의 결과를 통해 데이터를 유추하는 방법

- **Stored Procedure SQL Injection**

  데이터베이스에 저장된 프로시저를 이용하여 공격하는 방법

- **Mass SQL Injection**

  자동화된 스크립트를 사용하여 대량의 웹사이트를 대상으로 SQL Injection 공격을 시도하는 방법

## SQL Injection 대응방안 I

```sql
String sql = "insert into member(member_id, money) values(?, ?)";

# 강의 : JDBC 개발 - 등록 부분 참고
```

- **파라미터화된 쿼리 사용**

  SQL 쿼리에 사용자 입력 값을 직접 삽입하는 대신, 파라미터화된 쿼리를 사용하여 SQL Injection 공격을 방지한다.

- **Prepared Statement 구문사용**

  Prepared Statement 구문을 사용하면 사용자의 입력 값이 데이터베이스의 파라미터로 들어가기 전에 DBMS가 미리 컴파일 하여 실행하지 않고 대기한다. 그 후 사용자의 입력 값을 문자열로 인식하게 하여 공격쿼리가 들어간다고 하더라도, 사용자의 입력은 이미 의미 없는 단순 문자열 이기 때문에 전체 쿼리문도 공격자의 의도대로 작동하지 않는다.

## SQL Injection 대응방안 II

- 사용자의 입력 값에 대한 검증

  - 서버 단에서 `화이트리스트 기반`으로 검증해야 한다.

    블랙리스트 기반으로 검증하게 되면 수많은 차단리스트를 등록해야 하고, 하나라도 빠지면 공격에 성공하게 되기 때문이다.

  - 공백 대신 공격 키워드와는 `의미 없는 단어로 치환`한다.

    공백으로 치환하는 방법도 많이 쓰이는데, 이 방법도 취약한 방법이다. 공격자가 SESELECTLECT 라고 입력 시 중간의 SELECT가 공백으로 치환이 되면 SELECT 라는 키워드가 완성되게 된다. 즉, 공백 대신 공격 키워드와는 의미 없는 단어로 치환되어야 한다.

- Error Message 노출 금지

  공격자가 SQL Injection을 수행하기 위해서는 데이터베이스의 정보(테이블명, 컬럼명 등)가 필요하다.
  데이터베이스 에러 발생 시 따로 처리를 해주지 않았다면, `에러가 발생한 쿼리문과 함께 에러에 관한 내용을 반환`한다.

  여기서 테이블명 및 컬럼명 그리고 쿼리문이 노출이 될 수 있기 때문에, 데이터 베이스에 대한 오류발생 시 `사용자에게 보여줄 수 있는 페이지를 제작` 혹은 `메시지박스`를 띄우도록 하여야 한다.

### 참고

> 뤼튼 <br> [최근 해킹 이슈인 SQL 주입 공격(SQL Injection)이란?](https://blog.naver.com/PostView.naver?blogId=owlsystems2009&logNo=223010199135) <br> [SQL 인젝션 기초](https://www.bugbountyclub.com/pentestgym/view/52) <br> [SQL Injection 이란? (SQL 삽입 공격)](https://noirstar.tistory.com/264) <br>
