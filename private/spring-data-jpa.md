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

***

## 공통 인터페이스 주요 메서드
* `save(S)` : 새로운 엔티티는 저장하고 이미 있는 엔티티는 병합
* `delete(T)` : 엔티티 하나를 삭제 (내부에서 em.remove() 호출)
* `findById(ID)` : 엔티티 하나를 조회 (내부에서 em.find() 호출)
* `getOne(ID)` : 엔티티를 프록시로 조회 (내부에서 em.getReference() 호출)
* `findAll(...)` : 모든 엔티티 조회 (Sort나 Pageable 조건을 파라미터로 제공할 수 있음)  

> `S` : 엔티티와 그 자식 타입  
`T` : 엔티티  
`ID` : 엔티티의 식별자 타입  

***

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
#### 값 하나
```java
@Query("select m.username from Member m")
List<String> findUsernameList();
```
#### 임베디드 타입
```java
@Query("select m.address from Member m")
List<Address> findUserAddressList();
```
#### DTO 직접 조회
```java
@Query("select new study.datajpa.dto.MemberDto(m.id, m.username, t.name) from Member m join m.team t")
List<MemberDto> findMemberDto();
```  

***

## 반환 타입
스프링 데이터 JPA는 유연한 반환 타입을 지원한다.  

### 컬렉션
```java
List<Member> findListByUsername(String username);
```  
컬렉션으로 반환 타입을 지정했을때 결과가 없으면 **빈 컬렉션을 반환한다.**  

### 단건  
```java
Member findMemberByUsername(String username);
```  
단건으로 반환 타입을 지정하면 스프링 데이터 JPA 내부에서 JPQL의 `Query.getSingleResult()`를 호출한다.  
결과가 없으면 **null** (JPA에서는  `javax.persistence.NoResultException` 예외가 발생했었다.)  
결과가 2건 이상 이면 **`javax.persistence.NonUniqueResultException` 예외가 발생한다.**  
### 단건 Optional
```java
Optional<Member> findOptionalByUsername(String username);
```

***

## 페이징과 정렬
다음 조건으로 페이징과 정렬을 해보자
* 검색 조건 : 나이가 10살
* 정렬 조건 : 이름으로 내림차순
* 페이징 조건 : 첫 번째 페이지, 페이지당 보여줄 데이터는 3건  

### 순수 JPA 페이징과 정렬
```java
public List<Member> findByPage(int age, int offset, int limit) {
        return em.createQuery("select m from Member m where m.age = :age order by m.username desc", Member.class)
                .setParameter("age", age)
                .setFirstResult(offset)
                .setMaxResults(limit)
                .getResultList();
    }

public long totalCount(int age) {
    return em.createQuery("select count(m) from Member m where m.age = :age", Long.class)
            .setParameter("age", age)
            .getSingleResult();
}
```
```java
@Test
public void paging() throws Exception {

    //given
    memberJpaRepository.save(new Member("member1", 10));
    memberJpaRepository.save(new Member("member2", 10));
    memberJpaRepository.save(new Member("member3", 10));
    memberJpaRepository.save(new Member("member4", 10));
    memberJpaRepository.save(new Member("member5", 10));

    int age = 10;
    int offset = 0;
    int limit = 3;

    //when
    List<Member> members = memberJpaRepository.findByPage(age, offset, limit);
    long totalCount = memberJpaRepository.totalCount(age);

    //그러고..
    //페이지 계산 공식 적용...
    //totalPage 개수 구하기..
    //마지막 페이지..
    //최초 페이지..

    //then
    assertThat(members.size()).isEqualTo(3);
    assertThat(totalCount).isEqualTo(5);
}
```

### 스프링 데이터 JPA 페이징과 정렬

#### 페이징과 정렬 파라미터
* `org.springframework.data.domain.Sort` : 정렬 기능
* `org.springframework.data.domain.Pageable` : 페이징 기능 (내부에 Sort 포함)
#### 특별한 반환 타입
* `org.springframework.data.domain.Page` : 추가 count 쿼리 결과를 포함하는 
* `org.springframework.data.domain.Slice` : 추가 count 쿼리 없이 다음 페이지만 확인 가능 **(내부적으로 limit+1 조회)**
* `List` : 추가 count 쿼리 없이 결과만 반환  
```java
Page<Member> findByusername(String name, Pageable pageable); //count 쿼리 사용
Slice<Member> findByusername(String name, Pageable pageable); //count 쿼리 사용 안함
List<Member> findByusername(String name, Pageable pageable); //count 쿼리 사용 안함
List<Member> findByusername(String name, Sort sort); //count 쿼리 사용
```

#### 예시
```java
Slice<Member> findByAge(int age, Pageable pageable);
```  
```java
@Test
public void paging() throws Exception {

    //given
    memberRepository.save(new Member("member1", 10));
    memberRepository.save(new Member("member2", 10));
    memberRepository.save(new Member("member3", 10));
    memberRepository.save(new Member("member4", 10));
    memberRepository.save(new Member("member5", 10));

    int age = 10;
    PageRequest pageRequest = PageRequest.of(0, 3, Sort.by(Sort.Direction.DESC, "username"));

    //when
    Slice<Member> page = memberRepository.findByAge(age, pageRequest);
    //Slice<MemberDto> toMap = page.map(m -> new MemberDto(m.getId(), m.getUsername(), null));

    //then
    List<Member> content = page.getContent();
    assertThat(content.size()).isEqualTo(3);
    //assertThat(page.getTotalElements()).isEqualTo(5); Page에만 있는 메소드
    //assertThat(page.getTotalPages()).isEqualTo(2);
    assertThat(page.getNumber()).isEqualTo(0);
    assertThat(page.isFirst()).isTrue();
    assertThat(page.hasNext()).isTrue();
    for (Member member : content) {
        System.out.println("member = " + member);
    }
}
```
* `Pageable`은 인터페이스다. 보통 구현체로 `org.springframework.data.domain.PageRequest` 객체를 사용한다.
* `PageRequest` 생성자의 첫 번째 파라미터에는 현재 페이지를, 두 번째 파라미터에는 조회할 데이터 수를 입력한다.  
  추가로 정렬 정보도 파라미터로 사용가능하며 **페이지는 0부터 시작한다.**
* Page와 Slice 모두 map 기능을 제공하기 때문에 페이지를 유지하면서 엔티티를 DTO로 변환할 수 있다.
* 카운트 쿼리는 전체 데이터를 조회해야되기 때문에 굉장히 무겁다 그래서 다음과 같이 카운트 쿼리를 분리 할 수 있다.  
  ```java
  @Query(value = “select m from Member m join team t”,
         countQuery = “select count(m.username) from Member m”)
  Page<Member> findMemberAllCountBy(Pageable pageable);
  ```

***

## 벌크성 수정 쿼리

### 순수 JPA 벌크성 수정 쿼리
```java
public int bulkAgePlus(int age) {
    return em.createQuery("update Member m set m.age = m.age + 1 where m.age >= :age")
            .setParameter("age", age)
            .executeUpdate();
}
```
### 스프링 데이터 JPA 벌크성 수정 쿼리
```java
@Modifying(clearAutomatically = true)
@Query("update Member m set m.age = m.age + 1 where m.age >= :age")
int bulkAgePlus(@Param("age") int age);
```
* 벌크성 수정, 삭제 쿼리는 `@Modifying` 어노테이션을 사용해야한다. (사용하지 않으면 예외 발생)  
* `clearAutomatically = true` 옵션을 통해 벌크성 쿼리를 실행하고 나서 영속성 컨텍스트를 초기화 할 수 있다. (기본값 : false)  

***

## @EntityGraph
연관된 엔티티들을 SQL 한번에 조회하는 방법  
```java
//페치조인을 명시해서 사용하는 일반적인 방법
@Query("select m from Member m left join fetch m.team")
List<Member> findMemberFetchJoin();

//공통 메서드를 오버라이드
@Override
@EntityGraph(attributePaths = {"team"})
List<Member> findAll();

//JPQL + 엔티티 그래프
@EntityGraph(attributePaths = {"team"})
@Query("select m from Member m") //끼워넣기 가능
List<Member> findMemberEntityGraph();

//메서드 이름으로 쿼리에서도 가능
@EntityGraph(attributePaths = {"team"})
List<Member> findEntityGraphByUsername(@Param("username") String username);
```
* 사실상 페치 조인의 간편 버전
* left outer join 사용
* NamedEntityGraph라는 것도 있다.  

*** 

## JPA Hint
SQL 힌트가 아니라 JPA 구현체에게 제공하는 힌트  
```java
@QueryHints(value = @QueryHint(name = "org.hibernate.readOnly", value = "true"))
Member findReadonlyByUsername(String username);
```
```java
@Test
public void queryHint() throws Exception {

    //given
    Member member1 = new Member("member1", 10);
    memberRepository.save(member1);
    em.flush();
    em.clear();

    //when
    Member findMember = memberRepository.findReadonlyByUsername("member1");
    findMember.setUsername("member2");
    em.flush(); //update Query 실행 안됨
    //then
}
```
* `QueryHint의 readOnly`는 스냅샷을 만들지 않기 때문에 메모리가 절약됨  
* 참고로 `@Transaction(readOnly=true)`는 트랜잭션 커밋 시점에 flush를 하지 않기 때문에 dirty checking 비용이 들지 않아 cpu가 절약됨  
  스프링 5.1 버전 이후 부터는 `@Transaction(readOnly=true)`로 설정시 `QueryHint의 readOnly`까지 모두 동작함
  DTO 직접 조회시에는 스냅샷이 만들어지지 않음

```java
@QueryHints(value = { @QueryHint(name = "org.hibernate.readOnly",
                                 value = "true")},
                                 forCounting = true)
Page<Member> findByUsername(String name, Pagable pageable);
```
* `forCounting` : 반환 타입으로 Page 인터페이스를 적용하면 추가로 호출하는 count 쿼리도 쿼리 힌트 적용 (기본값 true)





