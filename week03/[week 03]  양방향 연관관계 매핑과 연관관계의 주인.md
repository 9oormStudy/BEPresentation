# 양방향 연관관계 매핑과 연관관계의 주인

![](https://velog.velcdn.com/images/gltdhd/post/8386f9c9-af45-4ebe-927d-b29b030f9c25/image.png)

회원(Member) 객체와 팀(Team) 객체가 있을 때, 회원은 하나의 팀(one)에 속하고 팀은 여러 회원(many)을 포함할 수 있다. 이런 경우에는 회원이 팀을 참조하고 **동시에** 팀도 회원을 참조하는 것이 양방향 연관관계이다.

**객체적 관점**에서 멤버와 팀 객체가 다른 객체를 참조하기 위해 각각 참조 변수(Member.team, Team.members)를 포함하면 양방향 객체 연관관계가 성립한다.

**DB 테이블 관점**에서는 **외래 키(Foreign Key) 하나로 양쪽 테이블 조인이 가능하다.** 
 ```sql
  /*테이블의 경우 외래 키(MEMBER.TEAM_ID) 하나로 양방향 연관관계를 가짐*/
  SELECT *
  FROM MEMBER M
  JOIN TEAM T ON M.TEAM_ID = T.TEAM_ID

  SELECT *
  FROM TEAM T
  JOIN MEMBER M ON T.TEAM_ID = M.TEAM_ID
  ```
따라서 데이터 베이스 테이블은 처음부터 양방향 관계이다.

양방향 객체 연관관계를 테이블 연관관계와 매핑한 결과는 다음과 같다.
```java
@Entity
public class Member {
    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;
    @Column(name = "USERNAME")
    private String username;
    
    // 연관관계 매핑
    @ManyToOne
    @JoinColumn(name = "TEAM_ID") 
    private Team team;
    ...
}

@Entity
public class Team {
    @Id
    @GeneratedValue
    @Column(name="TEAM_ID")
    private Long id;
    private String name;
    
    // 연관관계 매핑
    @OneToMany(mappedBy="team") ❗️
    private List<Member> members = new ArrayList<>();
    ...
}    
```



<br>

객체는 양방향 연관관계라기 보다는 **서로 다른 단방향 연관관계 2개**를 애플리케이션 로직으로 묶어 양방향인 것처럼 보인다.
- 회원 -> 팀 연관관계 1개 (단방향)
- 팀 -> 회원 연관관계 1개 (단방향)

반면 데이터베이스 테이블은 외래 키 하나로 양쪽이 서로 조인할 수 있으므로 **외래 키 하나만으로 양방향 연관관계**를 맺는다.
- 회원 <-> 팀 연관관계 1개 (양방향)

즉 엔티티를 양방향으로 설정하면 객체의 참조는 둘인데 외래키는 하나라서 둘 사이에 차이가 발생한다.

따라서 두 객체 연관관계 중에서 하나를 정해 테이블의 **외래 키를 관리하는 연관관계의 주인**을 설정해야 한다.

연관관계의 주인만이 데이터베이스 연관관계와 매핑되고 외래 키를 관리(등록, 수정, 삭제)할 수 있지만, **주인이 아닌 쪽은 오직 읽기만 가능하다.**

어떤 연관관계를 주인으로 정할지는 `mappedBy` 속성을 사용한다.

기본적으로 **외래 키가 있는 쪽이 외래키를 관리하는 연관관계 주인으로 설정해야 한다.**
여기서는 MEMBER 테이블에 외래키가 있으므로, Member.team 이 연관관계의 주인이 된다.
따라서 mappedBy의 값을 연관관계의 주인 필드로 설정한다 `mappedBy = team` 

![](https://velog.velcdn.com/images/gltdhd/post/e23b83f8-ddda-4b1e-9f93-80b3e56975a8/image.png)


```java
// 팀 생성
Team team = new Team();
team.setName("TeamA");
em.persist(team);

// 회원 저장
Member member = new Member();
member.setName("member1");
member.setTeam(team); // 연관관계 설정 member -> team

// 영속성 컨텍스트에 저장
em.persist(member);
```
연관관계의 주인에 값을 설정한 뒤 멤버 테이블을 조회한 결과는 다음과 같다.
<table><thead><tr><th>Member_ID (PK)</th><th>Name</th><th>Team_ID (FK)</th></tr></thead><tbody><tr><td>1</td><td>member1</td><td>1</td></tr></tbody></table>

JPA는 연관관계 매핑으로 팀의 식별자(기본키 PK)를 외래 키로 사용해서 적절한 등록 쿼리를 생성한다. 이는 단방향 연관관계에서의 결과와 똑같다.

Member.team 은 연관관계의 주인이므로 외래 키를 자유롭게 관리할 수 있다.
하지만 다음과 같은 입력값은 연관관계의 주인이 아니므로 외래 키 값에 영향을 주지 못하고, 무시된다.
```java
team.getMembers().add(member); // 주인이 아닌 곳에서 연관관계 설정 (무시됨)
```