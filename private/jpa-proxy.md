# JPA 프록시와 연관관계 관리
## 프록시
실제 클래스를 상속 받아서 만들어진 가짜 객체  
지연로딩의 핵심  

### 프록시 구조
* 프록시 객체는 실제 객체의 참조(target)를 보관
* 프록시 객체를 호출하면 프록시 객체는 실제 객체의 메소드 호출

### 프록시 객체의 초기화
1. 초기화가 안된 프록시 객체에서 getName()등을 호출
2. target이 null 이므로 **영속성 컨텍스트**에 초기화 요청
3. 영속성 컨텍스트가 해당 엔티티 DB 조회후 영속한뒤 target에 넣어줌
4. target.getName()을 반환

### em.find() vs em.getReference()
* **find**  
  데이터베이스를 통해서 실제 엔티티 객체 조회
* **getReference**  
  데이터베이스 조회를 미루는 가짜(프록시) 엔티티 객체 조회  



### 프록시의 특징
* 프록시 객체는 처음 사용할 때 한 번만 초기화
* 프록시 객체를 초기화 할 때, 프록시 객체가 실제 엔티티로 바뀌는 것은 아님, 초기화되면 프록시 객체를 통해서 실제 엔티티에 접근 가능
  ```java
  Member refMember = em.getReference(Member.class, member.getId);
  System.out.println(refMember.getClass()); //class x.Member$HibernateProxy$xxxxxxx
  refMember.getName();
  System.out.println(refMember.getClass()); //class x.Member$HibernateProxy$xxxxxxx
  ```
* 프록시 객체는 원본 엔티티를 상속받음, 따라서 **타입 체크시 주의**해야함(== 대신 instanceof 사용)
  ```java
  Member member1 = em.find(Member.class, member1.getId());
  Member member2 = em.getReference(Member.class, member2.getId);
  System.out.println(member1.getClass() == member2.getClass()); //false
  System.out.println(member2.getClass() instanceof Member);     //true
  ```
* 영속성 컨텍스트에 찾는 엔티티가 이미 있으면 **em.getReference()를 호출해도 실제 엔티티 반환, 그 반대도 마찬가지**
  ```java
  Member findMember = em.find(Member.class, member.getId());
  System.out.println(findMember.getClass());                         //class x.Member
  Member refMember = em.getReference(Member.class, member.getId);
  System.out.println(refMember.getClass());                          //class x.Member
  System.out.println(findMember.getClass() == refMember.getClass()); //true
  ```  
  getReference후에 find를 호출함에도 프록시가 반환된다!  
  ```java
  Member refMember = em.getReference(Member.class, member.getId());
  System.out.println(refMember.getClass());                          //class x.Member$HibernateProxy$xxxxxxx
  Member findMember = em.find(Member.class, member.getId);
  System.out.println(findMember.getClass());                         //class x.Member$HibernateProxy$xxxxxxx
  System.out.println(findMember.getClass() == refMember.getClass()); //true
  ```
  > JPA는 객체패러다임을 준수하기 위해 한 영속성 컨텍스트에서 관리되는 엔티티는 같은 엔티티라면 == 비교시 무조건 true가 되도록 동작해야하기 때문  
* 영속성 컨텍스트의 도움을 받을 수 없는 준영속 상태일 때, 프록시를 초기화하면 문제 발생
  ```java
  Member refMember = em.getReference(Member.class, member.getId());
  em.detach(refMember); //or em.close() em.clear()
  refMember.getName();  //throw org.hibernate.LazyInitializationException
  ```
### 프록시 유틸 메소드
* **프록시 인스턴의 초기화 여부 확인**  
  emf.getPersistenceUnitUtil().isLoaded(entity)
* **프록시 강제 초기화**  
  org.hibernate.Hibernate.initialize(entity)  
> JPA 표준은 강제 초기화가 없어서 entity.getName()식으로 해줘야됨  

## 즉시 로딩과 지연 로딩
JPA에서는 getReference를 통해 프록시를 직접 사용하지 않는다.  
대신 `즉시 로딩`과 `지연 로딩`이라는 전략을 설정해 사용한다.
### 지연 로딩
연관관계 매핑 에노테이션에 fetch를 `FetchType.LAZZ`로 설정하면  
해당 엔티티는 지연 로딩이 된다.  
지연 로딩으로 설정하면 조회시 프록시로 반환되고 실제 사용이 이루어져야 DB에서 조회된다.  
```java
@Entity
public class Member {

  @Id
  @GeneratedValue
  private Long id;
  
  @Column(name = "USERNAME")
  private String name;
  
  @ManyToOne(fetch = FetchType.LAZY) //**
  @JoinColumn(name = "TEAM_ID")
  private Team team;
  ..
}
```
```java
Member findMember = em.find(Member.class, 1L);  //member는 DB에서 조회하고 team은 프록시로 반환함
System.out.println(findMember.getClass());      //class x.Member
Team team = findMember.getTeam();               //여기선 초기화 안됨
System.out.println(team.getClass());            ////class x.Team$HibernateProxy$xxxxxxx
team.getName();                                 //초기화됨

```

### 즉시 로딩
연관관계 매핑 에노테이션에 fetch를 `FetchType.EAGER`로 설정하면  
해당 엔티티는 즉시 로딩이 된다.
즉시 로딩으로 설정하면 처음 조회시 join을 통해 관련된 모든 엔티티를 가져온다.  
> JPA 구현체는 가능하면 조인을 사용해 SQL 한번에 함께 조회한다.  
일부는 개별 SQL로 조회하는듯 하다.  

> **주의사항**  
  무조건 처음에는 지연 로딩만 사용해야한다.  
> * 예상못한 SQL이 발생할 수 있다.  
    연관된 테이블이 여러개면 전부 조인해서 가져온다.  
> * **즉시 로딩은 JPQL에서 N+1 문제를 일으킨다.**  
    em.createquery("select m from Member").getResultList();  
    위 JPQL은 select * from Member 라는 SQL로 번역이되어서 DB로 보내지게 된다.  
    그러면 처음에는 Member만 가져오게 되는데 이때 Team이 Eager이므로 즉시로딩을 만족하기 위해  
    member 객체 별로 team을 찾기 위한 select 쿼리가 발생한다.  
> * **@XToOne은 기본값이 즉시 로딩이기 때문에 필수록 LAZY로 설정해야한다.**  
  

  
