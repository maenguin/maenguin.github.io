# 객체지향 쿼리 언어 (JPQL)
* [경로 표현식](#경로-표현식)
* [다형성 쿼리](#다형성-쿼리)
* [엔티티 직접 사용](#엔티티-직접-사용)
* [Named 쿼리](#Named-쿼리)
* [벌크 연산](#벌크-연산)
## 경로 표현식
. (점)을 찍어 객체 그래프를 탐색하는 것
* 상태 필드
* 연관 필드
  * 단일 값 연관 필드
  * 컬렉션 값 연관 필드
### 상태 필드
단순히 값을 저장하기 위한 

***

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

***

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

***

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







