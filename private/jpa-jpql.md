# 객체지향 쿼리 언어 (JPQL)
* [기본 문법과 기능](#기본-문법과-기능)
* [경로 표현식](#경로-표현식)
* [다형성 쿼리](#다형성-쿼리)
* [엔티티 직접 사용](#엔티티-직접-사용)
* [Named 쿼리](#Named-쿼리)
* [벌크 연산](#벌크-연산)

## 기본 문법과 기능
### JPQL 문법
```sql
select_절
from_절
[where_절]
[groupby_절]
[having_절]
[orderby_절]

update_절 [where_절]
delete_절 [where_절]
```
* 엔티티와 속성은 대소문자 구분O (Member, age)
* JPQL 키워드는 대소문자 구분X (SELECT, FROM, where)
* 테이블 이름이 아닌 엔티티 이름 사용
* **별칭은 필수** (as는 생략 가능)  

### TypeQuery & Query
#### TypeQuery  
반환 타입이 명확할 때 사용  
```java
TypeQuery<Member> query = em.createQuery("select m from member m", Member.class);
```
#### Query
반환 타입이 명확하지 않을 때 사용
```java
Query query = em.createQuery("select m.username, m.age from Member m");
```

### 결과 조회 API
#### query.getResultList()
* **결과가 하나 이상일 때**, 리스트 반환
* 결과가 없으면 빈 리스트 반환
#### query.getSingleResult()
* **결과가 정확히 하나**, 단일 객체 반환
* 결과가 없으면 `javax.persistence.NoResultException` throw
* 둘 이상이면 `javax.persistence.NonUniqueResultException` throw
> 스프링 데이터 JPA에서는 한번 감싸서 null이나 optional 반환

### 파라미터 바인딩
#### 이름 기준 바인딩
```sql
em.createQuery(select m from Member m where m.username = :uuu)
  .setParameter("uuu", "회원1");
```
#### 위치 기준 바인딩
```sql
em.createQuery(select m from Member m where m.username = ?1)
  .setParameter(1, "회원1");
```

### 프로젝션
select절에 조회할 대상을 지정하는 것
#### 엔티티 프로젝션
조회할 대상에 엔티티를 지정하는 것으로 조회된 엔티티는 영속성 컨텍스트에 관리된다. 타입 필수
```sql
em.createQuery("select m from Member m", Member.class)
em.createQuery("select m.team from Member m", Team.class)
em.createQuery("select m.orders from Member m", Order.class)
```
#### 임베디드 타입 프로젝션
```sql
em.createQuery("select m.address from member m", Address.class)
```
#### 스칼라 타입 프로젝션
```sql
em.createQuery("select m.username, m.age from Member m")
```
**위 예시처럼 반환타입이 명확하지 않을때는 어떻게 처리해야 할까?**
#### Query 타입으로 조회


****************************************************************

## 경로 표현식
. (점)을 찍어 객체 그래프를 탐색하는 것
* 상태 필드
* 연관 필드
  * 단일 값 연관 필드
  * 컬렉션 값 연관 필드
### 상태 필드
단순히 값을 저장하기 위한 필드 (ex: m.username)  
경로 탐색의 끝으로서 더이상 .을 찍어서 탐색 불가  
```sql
[JPQL]
select m.username, m.age from Member m

[SQL]
select m.username,
       m.age
from   member m
```

### 단일 값 연관 필드
연관관계를 위한 필드 중 @ManyToOne, @OneToOne로 매핑한 엔티티 필드 (ex: m.team)  
**묵시적 내부 조인이 발생**  
탐색 가능 (ex: m.team.name)  
```sql
[JPQL]
select o.member from Order o

[SQL]
select m.*
from   orders o
       inner join member m 
               on o.member_id = m.id
```

### 컬렉션 값 연관 필드
연관관계를 위한 필드 중 @OneToMany, @ManyToMany로 매핑한 컬렉션 필드 (ex: m.orders)  
**묵시적 내부 조인이 발생**  
**탐색 불가**, 하지만 from 절에서 명시적 조인을 통해 별칭을 얻으면 탐색 가능해짐   
```sql
[JPQL]
select m.orders.name from member m //불가능
select o.name from member m join orders o //명시적 조인으로 탐색

select m.orders from member m

[SQL]
select o.*
from   member m
       inner join orders o 
               on m.id = o.member_id
```

### 명시적 조인과 묵시적 조인
* 명시적 조인  
  * **join 키워드를 직접 사용**하는 조인  
  * select m from member m **join m.team t**
* 묵시적 조인  
  * **경로 표현식에 의해 묵시적으로 SQL 조인이 발생**하는 조인(내부 조인만 가능)  
  * select **m.team** from member m  
  
**가급적 묵시적 조인 대신에 명시적 조인을 사용해야 한다.**  
묵시적 조인은 조인이 일어나는 상황을 한눈에 파악하기 어렵다.  

**************************************************************

## 다형성 쿼리
Item을 상속받은 Book과 Movie 엔티티가 있다고 가정해보자  
### TYPE
조회 대상을 특정 자식으로 한정
```sql
[JPQL]
select i from item i where type(i) in (Book, Movie)

[SQL]
select i.*
from   item i
where  i.dtype in ('B', 'M')
```
### TREAT(JPA 2.1)
자바의 타입 캐스팅 처럼 부모 타입을 특정 자식 타입으로 다룸  
from, where, select(하이버네이트 지원)에서 사용가능
```sql
[JPQL]
select i from item i where treat(i as Book).author = 'kim'

방언 및 테이블 전략에 맞춰서 적절한 쿼리가 나간다

[SQL] 
select i.*
from   item i
where  i.dtype = 'B' and i.author = 'kim' 
```

**********************************************************************

## 엔티티 직접 사용
JPQL에서 엔티티를 직접 사용하면 SQL에서 해당 엔티티의 기본 키 값을 사용한다.  
```sql
[JPQL]
select count(m.id) from Member m (엔티티의 id 사용)
select count(m) from Member m    (엔티티 직접 사용)

둘다 아래의 SQL로 실행된다.

[SQL]
select count(m.id) as cnt
from   Member m
```  
  
파라미터로 전달할 경우
```sql
[JPQL]
select m from Member m where m.id = :memberId (엔티티의 id 사용)
select m from Member m where m = :member      (엔티티 직접 사용)

둘다 아래의 SQL로 실행된다.

[SQL]
select m.*
from   Member m
where  m.id = ?
```  
  
외래키의 경우
```sql
[JPQL]
select m from Member m where m.team.id = :teamId  (엔티티의 id 사용)
select m from Member m where m.team = :team       (엔티티 직접 사용)

둘다 아래의 SQL로 실행된다.

[SQL]
select m.*
from   Member m
where  m.team_id = ?
```  

*********************************************************************

## Named 쿼리
* 미리 정의해서 이름을 부여해두고 사용하는 JPQL
* 정적 쿼리이기 때문에 동적쿼리 불가
* 어노테이션이나 XML로 사용가능
* 애플리케이션 로딩 시점에 초기화 후 재사용
  SQL로 미리 번역해두기 때문에 코스트 절약
* **애플리케이션 로딩 시점에 쿼리를 검증**
  SQL 오류를 방지하는데 굉장히 효과적

### @NamedQuery & em.createNamedQuery()
```java
@Entity
@NamedQuery(
        name = "Member.findByName", 
        query = "select  m from Member m where m.name = :name")
public class Member {
    ...
}
```
```java
em.createNamedQuery("Member.findByName", Member.class)
  .setParameter("name", "회원1")
  .getResultList();
```
> 스프링 데이터 JPA에서는 @Query를 통해 Repository 인터페이스에서 더 깔끔하게 네임드쿼리를 사용할 수 있다.  

****************************************************************************************************

## 벌크 연산
모든 Member의 나이를 20으로 바꿔야 된다고 가정해보자  
JPA 변경 감지 기능으로 해결하려면 너무 많은 SQL이 실행된다.  
그래서 JPA는 **한번의 쿼리로 여러 테이블의 로우를 변경**할 수 있는 기능을 제공한다.  

### executeUpdate()
* update, delete 지원
* 하이버네이트에서는 insert into..select 지원
```java
int resultCount = em.createQuery("update Member p set m.age = 20", Member.class)
                    .executeUpdate(); //업데이트 된 엔티티의 수를 반환한다.
```
### 벌크 연산 주의사항
벌크 연산은 영속성 컨텍스트를 무시하고 데이터베이스에 직접 쿼리한다.  
그렇기 때문에 영속성 컨텍스트와 실제 데이터베이스 사이의 데이터 정합성 문제가 발생할 수 있다.  
그렇기 때문에  
* 영속성 컨텍스트에 엔티티가 들어가기전에 벌크 연산을 먼저 수행하거나
* 벌크 연산 수행 후 영속성 컨텍스트를 초기화 해야한다.
  ```java
  int resultCount = em.createQuery("update Member p set m.age = 20", Member.class)
                    .executeUpdate();
  //em.clear(); 영속성 컨텍스트를 초기화 하지 않으면
  
  Member member1 = em.find(Member.class, 1L);
  member.getAge(); // 0 출력
  ```







