

# Java Persistence API (JPA)

## 소개
자바 진영에서 제공하는 ORM 기술이다.
말그대로 객체와 데이터베이스의 릴레이션을 맵핑해주며
개발자가 직접 쿼리를 작성해 데이터를 조작하지 않고 JPA가 대신 쿼리를 생성한다.


## 등장배경

### 객체 지향과 관계형 모델의 대립
자바와 관계형 데이터베이스를 이용해 개발을 한다고 가정해보자(대부분 이 경우 일것이다.)
이 경우 자바의 객체 지향과 관계형 모델의 패러다임 불일치가 발생한다.
그로인해 객체 모델을 관계형 모델로 또 관계형 모델을 객체 모델로 변환하는 번거로운 과정이 필요하고 중복코드를 양산하며 유지보수를 힘들게하는 원인이 되었다.

## 정보

* JPA는 Java aplication과 JDBC사이에서 동작한다.
* JPA는 JDBC의 Wrapper 기술인셈
* JPA는 인터페이스의 모음이다.
* 구현체로는 하이버네이트, EclipseLink, DataNucleus가 있고 JPA 2.1 표준 명세를 토대로 구현되었다.

## 역사

1. EJB - 엔티티 빈(자바 표준)
최초의 Java ORM 기술, 실용성과 성능의 한계로 인해 도태됨
2. 하이버네이트 (오픈 소스)
게빈 킹이 EJB의 한계를 느끼고 직접 ORM framework를 개발
3. JPA (자바 표준)
자바진영에서 하이버네이트를 자바 ORM 표준으로 정립

## 왜 사용해야 하는가?

### SQL 중심적인 개발에서 객체 중심으로 개발

### 생산성
기존의 복잡한 처리 과정을 거치지 않고 단 몇줄의 코드로 CRUD를 수행할 수 있다.
### 유지보수
기존에는 객체의 명세를 변경하면 관련된 SQL을 전부 찾아 변경해야 했지만 JPA가 모두 처리해준다.
### 패러다임 불일치 해결
1. 상속
객체의 상속관계의 경우 테이블을 슈퍼타입과 서브타입 관계로 흉내낼 수 는 있지만 결국 SQL은 테이블별로 나눠서 작성해야한다.
하지만 JPA는 이를 해결해준다.
2. 연관관계
3. 객체 그래프 탐색
4. equal
### 성능

### 데이터 접근 추상화와 벤더 독립성

### 표준

## JPA 구동 방식

1. Persistence 클래스가 설정 정보(META-INF/persistence.xml or application.yml)를 읽고
2. EntityManagerFactory를 생성한다.
3. EntityManagerFactory는 상황에 맞게 EntityManager들을 생성한다.

아래는 스프링없이 순수 java와 jpa만으로 동작하는 코드이며 `Member` 객체를 하나 생성한 뒤 저장하는 예제이다.  
```java
public static void main(String[] args) {

    EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");

    EntityManager em = emf.createEntityManager();

    EntityTransaction tx = em.getTransaction();
    tx.begin();

    try {
      Member member = new Member();
      member.setId(1L);
      member.setName("Member1");

      em.persist(member);

      tx.commit();
    } catch (Exception e) {
      tx.rollback();
    } finally {
      em.close();
    }

    emf.close();

  } 
```
코드를 분석해보면, `Persistence.createEntityManagerFactory`에 `persistence-unit name`을 넘겨서 `persistence.xml`에서 설정한 정보를 불러와 `EntityMangerFactory`를 생성하고 있다.  
그 후 생성한 `EntityMangerFactory`를 통해 `EntityManger`를 생성하고 `em.persist(member)`를 통해 member를 저장했다.  

코드를 자세히 보면 `EntityTransaction`을 통해 트랜잭션을 걸고 있는데 이는 **Jpa의 모든 데이터 변경은 무조건 트랜잭션 안에서 동작해야하기 때문이다.**  
그래서 아래와 같은 양식이 꼭 필 수 로 들어가게 된다.
```java
EntityTransaction tx = em.getTransaction();
tx.begin();

//...

tx.commit();
```
> Maven이 아니라 Gradle을 사용하는 경우 엔티티 클래스가 자동으로 인식되지 않는다.  
따로 persistence.xml에 \<class>\</clase>로 엔티티를 추가해주어야한다. 

> EntityManagerFactory는 DB당 하나만 생성해서 애플리케이션 전체에서 공유해야한다.

> EnitityManager는 DB의 커넥션 하나를 물고 있다고 생각하면 된다.  
그렇기 때문에 client에서 요청할때 마다 생성하고 트랜잭션이 끝나면 무조건 close를 해주어야한다.  
쓰레드간에 공유도 하면 안된다.




## JPQL
JPA에서 제공하는 SQL을 추상화한 객체 지향 쿼리 언어  
id를 통한 단건 조회를 제외하고 JPA에서는 `JPQL`을 이용해 조회를 해야한다.
```java
List result = em.createQuery("select m FROM Member as m")
                .setFirstResult(5)
                .setMaxResults(8)
                .getResultList();
```
코드를 보면 익히 알던 SQL 쿼리와 유사하면서도 조금 다르다.  
여기서 Member는 테이블이 아니라 객체를 의미하는데 이는 **JPQL은 엔티티 객체를 대상으로 쿼리**를 만들기 때문이다.  
데이터베이스 테이블을 대상으로 쿼리를 만드는 SQL과 대조적이라 할 수 있다.  

객체를 대상으로 쿼리를 만듬으로서 특정 DB에 종속되지 않는다.  
그래서 DB를 변경한다고 해도 JPQL 변경없이 방언만 변경하면 해당 DB에 맞는 쿼리가 만들어진다.

