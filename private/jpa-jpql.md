# 객체지향 쿼리 언어 (JPQL)

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

## Named 쿼리
* 미리 정의해서 이름을 부여해두고 사용하는 JPQL
* 정적 쿼리이기 때문에 동적쿼리 불가
* 어노테이션이나 XML로 사용가능
* 애플리케이션 로딩 시점에 초기화 후 재사용
  SQL로 미리 번역해두기 때문에 코스트 절약
* **애플리케이션 로딩 시점에 쿼리를 검증**
  SQL 오류를 방지하는데 굉장히 효과적

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
> 스프링 데이터 JPA에서는 @Query를 통해 Repository 인터페이스에서 더 깔끔하게 네임드쿼리르 사용할 수 있다.  


