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

[SQL]
select i.*
from   item i
where  i.dtype = 'B' and i.author = 'kim'
```
