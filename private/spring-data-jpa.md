# 스프링 데이터 JPA


## 공통인터페이스 분석


### JpaRepository
`spring-data-jpa` 라이브러리의  
`org.springframework.data.jpa.respository` 패키지에 있는 인터페이스이다.  
Jpa에 특화된 인터페이스라고 보면된다.  
`PagingAndSortingRepository`와 `QueryByExampleExecuor` 인터페이스를 상속받았다.  


### PagingAndSortingRepository
`spring-data-commons` 라이브러리의  
`org.springframework.data.respository` 패키지에 있는 인터페이스이다.  
Jpa와 Rdb에 특화되어 있는 위와는 다르게 공통적인 속성으로 정의되어 있다.  
`CrudeRepository` 인터페이스를 상속받았다.  

### CrudeRepository
패키지는 위와 같다.  
기본적인 CRUD 기능이 정의되어 있다.  
`Repository` 인터페이스를 상속받았다.  

### Repository
패키지는 위와 같다.  
아무런 기능이 정의되어 있지 않고 마커 인터페이스 역할을 한다.  
최상단의 인터페이스로서 스프링 컴포넌트 스캔을 용이하게 해준다.  

## 공통 인터페이스 주요 메서드
* `save(S)` : 새로운 엔티티는 저장하고 이미 있는 엔티티는 병합
* `delete(T)` : 엔티티 하나를 삭제 (내부에서 em.remove() 호출)
* `findById(ID)` : 엔티티 하나를 조회 (내부에서 em.find() 호출)
* `getOne(ID)` : 엔티티를 프록시로 조회 (내부에서 em.getReference() 호출)
* `findAll(...)` : 모든 엔티티 조회 (Sort나 Pageable 조건을 파라미터로 제공할 수 있음)  

> `S` : 엔티티와 그 자식 타입  
`T` : 엔티티  
`ID` : 엔티티의 식별자 타입  


## 쿼리 메소드 기능
공통 인터페이스에 없는 메소드를 구현하려면?  

### 메소드 이름으로 쿼리 생성
**스프링 데이터 JPA는 메소드 이름을 분석해서 JPQL을 생성하고 실행하는 기능을 제공한다.**  
이름과 나이를 기준으로 회원을 조회하는 메소드를 구현한다고 가정하면  
순수 JPA의 경우 다음과 같이 직접 코드를 구현해야한다.  
```java
[순수 JPA]
public List<Member> findByUsernameAndAgeGreaterThan(String username, int age) {
    return em.createQuery("select m from Member m where m.username = :username and m.age > :age", Member.class)
            .setParameter("username", username)
            .setParameter("age", age)
            .getResultList();
}
```  
하지만 스프링 데이터 JPA를 사용할 경우 메소드 이름만 규칙에 맞게 지어주면 직접 구현하지 않아도 스프링 데이터 JPA가 자동으로 JPQL을 생성하고 실행한다.  
```java
[스프링 데이터 JPA]
List<Member> findByUsernameAndAgeGreaterThan(String username, int age);
```
#### 메소드 이름 규칙  
* `조회` : find...By, read...By, query...By, get...By
* `count` : count...By (반환타입: `long`)
* `exists` : exists...By (반환타입: `boolean`)
* `삭제` : delete...By, remove...By (반환타입: `long`)
* `distinct` : findDistinctBy
* `limit` : findFirst3, findFirst, findTop, findTop3  

**공식 문서 참고**  
[필터 조건](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods.query-creation)  
[qeury-creation](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories.query-methods.query-creation)  
[limit-query-result](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories.limit-query-result)  
  
> **참고**  
엔티티의 필드명이 변경되면 인터페이스에 정의한 메서드 이름도 꼭 함께 변경해야 된다.  
그렇지 않으면 애플리케이션 실행 시점에 오류가 발생한다.  
(로딩 시점에 오류인지 할 수 있는게 큰 장점)  

### JPA NamedQuery
스프링 데이터 JPA에서도 순수 JPA의 NamedQuery를 호출할 수 있다.  
```java
@NamedQuery(name = "Member.findByUsername", 
           query = "select m from Member m where m.username = :username")
public class Member {
    ...
}
```
```java
[순수 JPA]
public List<Member> findByUsername(String username) {
    return em.createNamedQuery("Member.findByUsername", Member.class)
            .setParameter("username", username)
            .getResultList();
}
```  
스프링 데이터 JPA에서는 `@Query`를 사용해 미리 정의된 네임드쿼리를 호출할 수 있는데  
다음과 같이 `@Query`를 생략해도 네임드쿼리가 호출된다.  
왜냐하면 스프링 데이터 JPA는 `도메인 클래스명.메소드명`으로 먼저 네임드쿼리가 존재하는지 확인하기 때문이다.(없으면 메서드 이름으로 쿼리 생성 전략을 사용)    
파라미터 바인딩을 위해 `@Param`을 사용한다.  
```java
[스프링 데이터 JPA]
//@Query(name = "Member.findByUsername")
List<Member> findByUsername(@Param("username") String username);
```

### 리포지토리 메소드에 쿼리 정의하기
네임드쿼리를 직접 등록해서 사용하는 일은 드물다.  
대신 **`@Query`를 사용해서 리포지토리 메소드에 쿼리를 직접 정의한다.**  
```java
@Query("select m from Member m where m.username = :username and m.age = :age")
List<Member> findUser(@Param("username") String username, @Param("age") int age);
```  
* 이름없는 Named 쿼리
* JPA Named 쿼리처럼 애플리케이션 실행 시점에 문법 오류를 발견할 수 있다.  
* 메소드 이름으로 쿼리 생성 기능은 파라미터가 증가하면 이름이 매우 지저분해지기 때문에 @Query를 자주 사용

### 값, DTO 조회하기








