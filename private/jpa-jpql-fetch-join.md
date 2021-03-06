# 페치 조인 (fetch join)

## 연관된 엔티티나 컬렉션을 SQL 한번에 가져오고 싶다
다대일 관계인 Order과 Member 엔티티가 있다고 하자.  
Order를 가져오면서 Member 정보도 같이 한번의 쿼리로 가져오고 싶을땐 어떻게 해야될까?  
  
* **단순히 Order를 가져와서 안에 Member가 있길 기대해보자**
  ```java
  em.createQuery("select o FROM Order o", Order.class).getResultList();
  ```
  ```sql
  select o.* 
  from   orders o 
  ```
  하지만 위 쿼리는 EAGER일 경우에 Member를 조회하는 쿼리가 따로 나가고  
  LAZY일 경우에 Member를 사용할 때 쿼리가 호출될 것이다.  
  둘다 N + 1 문제를 발생시키며 원하는 결과가 아니다.  
  
* **이번에는 조인을 해서 한번에 가져오도록 해보자**  
  ```java
  em.createQuery("select o FROM Order o join o.member m", Order.class).getResultList();
  ```
  ```sql
  select o.* 
  from   orders o 
         inner join member m 
                 on o.member_id = m.member_id 
  ```
  그러나 이 경우 역시 조인만 할 뿐 Order만 가져오며 위 경우와 같은 효과가 발생한다.    
  
* **그렇다면 select에 Member를 가져오게끔 표시하면 되지 않을까?**
  ```java
  em.createQuery("select o, m FROM Order o join o.member m").getResultList();
  ```
  ```sql
  select o.*,
         m.*
  from   orders o 
         inner join member m 
                 on o.member_id = m.member_id 
  ```
  결과적으로는 Memeber 데이터 까지 같이 가져온다.  
  하지만 이는 Type정보를 넣지 않았기 때문에 가능했다.  
  추가로 매핑작업을 해줘야 한다.  
  
그렇다면 어떻게 해야하는가!  
바로 **fetch join**을 사용하면된다. 
```java
em.createQuery("select o FROM Order o join fetch o.member m", Order.class).getResultList();
```
```sql
select m.*, 
       o.* 
from   orders o 
       join member m 
         on o.member_id = m.member_id 
```
Order와 연관된 Member까지 쿼리 한번으로 가져와서 Order 타입으로 받은걸 볼 수 있다.  


> **EAGER & em.find()**  
    fetch 전략을 EAGER로 한 상태에서 em.find()를 사용하면 단일 엔티티에 대하여 한번에 연관된 엔티티까지 가져올 수 있다.  
    하지만 EAGER를 사용해야되고 단일 조회만 가능하다.  
>   ```java
>   em.find(Order.class,1L);
>   ```
>   ```sql
>   select m.*, 
>          o.* 
>   from   orders o 
>          left join member m 
>                 on o.member_id = m.member_id 
>   where o.order_id = 1
>   ```  

## 페치 조인이란  
* 연관된 엔티티나 컬렉션을 **SQL 한번에 함께 조회**하는 기능
* JPQL에서 **성능 최적화**를 위해 제공
* SQL 조인 종류는 아니고 JPQL고유의 기능
* 사용법  
  [ left [outer] | inner ] join fetch 조인경로 

## 컬렉션 페치 조인
다대일뿐만 아니라 일대다도 페치 조인을 사용할 수 있다.  
  
양방향 연관관계가 설정되어 있다 가정하고 Member를 가져올때 연관된 Order 목록들을 가져오도록 해보자
```java
List<Member> members = em.createQuery("select m FROM Member m join fetch m.orders", Member.class).getResultList();
```
```sql
select m.*, 
       o.* 
from   member m 
       inner join orders o 
               on m.member_id = o.member_id 
```
데이터베이스에 날려진 SQL을 보면 이상없이 데이터를 가져온것 같지만 한가지 주의사항이 있다.  
일대다 조인시에는 **데이터가 부풀려지기 때문에 Member 컬렉션에 담는 과정에서 동일한 값이 들어갈 수 있다.**  
Member를 기준으로 조인을 걸게되면 아래 결과 테이블 처럼 연관된 Order 만큼 Member 튜플이 생성이 되는걸 볼 수 있다.  
|id|name|id|member_id|name|
|:---:|:---:|:---:|:---:|:---:|
|1|회원1|1|1|주문1|
|1|회원1|2|1|주문2|
```java
for(Member member : members) {
 System.out.println(member.getName() + " " + member);
 for (Order order : member.getOrders()) {
 System.out.println(“-> " + order.getName()+ " " + order);
}
/*
회원1 Member@0x100
-> 주문1 Order@0x200
-> 주문2 Order@0x300
회원1 Member@0x100
-> 주문1 Order@0x200
-> 주문2 Order@0x300
회원2 Member@0x400
-> 주문3 Order@0x500
-> 주문4 Order@0x600
*/
```
## 페치 조인과 DISTINCT
일대다 페치조인시 중복된 결과를 제거하기 위해 JPQL에서도 `DISTINCT` 명령어를 제공한다.  
JPQL의 DISTINCT는 2가지 기능을 제공
1. SQL에 DISTINCT를 추가
2. 애플리케이션에서 엔티티 중복 제거
```java
List<Member> members = em.createQuery("select distinct m FROM Member m join fetch m.orders", Member.class).getResultList();
```
```sql
select distinct m.*, 
                o.* 
from   member m 
       inner join orders o 
               on m.member_id = o.member_id 
```
보면 SQL에 DISTINCT를 추가한다. 하지만 컬럼별로 데이터가 완전히 같지 않아서 중복제거에는 실패한다.  
그래서 애플리케이션에서 중복 제거를 시도한다.  
**같은 식별자**를 가진 Member 엔티티를 제거한다.  
```java
for(Member member : members) {
 System.out.println(member.getName() + " " + member);
 for (Order order : member.getOrders()) {
 System.out.println(“-> " + order.getName()+ " " + order);
}
/*
회원1 Member@0x100
-> 주문1 Order@0x200
-> 주문2 Order@0x300
회원2 Member@0x400
-> 주문3 Order@0x500
-> 주문4 Order@0x600
*/
```

## 페치 조인과 일반 조인의 차이
* JPQL에서의 조인도 결과를 반환할 때 연관관계를 고려하지 않음
* 단지 select 절에 짖어한 엔티티만 조회
* 페치 조인을 사용하면 연관된 엔티티도 함께 조회함(즉시 로딩)
* **페치 조인은 객체 그래프를 SQL 한번에 조회하는 개념**

## 페치 조인의 특징과 한계
* **페치 조인 대상에는 별칭을 줄 수 없다.**  
  JPA 설계사상에서 객체그래프를 탐색한다는 것은 연관된 객체 전부를 가져온다는 전제를 가지고 있다.  
  아래 코드와 같이 별칭을 줘서 Order를 전부 가져오지 않고 걸러서 가져온다면 데이터 정합성에 문제가 발생할 수 있다.
  ```java
  //하이버네이트는 별칭가능
  em.createQuery("select m FROM Member m join fetch m.orders o where o.price > 1000", Member.class).getResultList();
  //이렇게 페치 조인을 여러단계로 해야되는 경우를 제외하고는 페치 조인 대상의 별칭은 사용하지 않는게 좋다.
  em.createQuery("select m FROM Member m join fetch m.orders o join fetch o.delivery ", Member.class).getResultList();
  ```
* **둘 이상의 컬렉션은 페치 조인 할 수 없다.**  
  일대다의 데이터 부풀리기 처럼 더 많은 부풀리기가 발생해서 정합성 문제가 발생할 수 있다.  
* **컬렉션을 페치 조인하면 페이징 API를 사용할 수 없다.**  
  일대일, 다대일 같은 단일 값 연관 필드들은 페치 조인을 해도 페이징이 가능하다.  
  하지만 다대일의 경우 데이터 부풀림 때문에 페이징시 의도와는 다르게 동작할 수 있다.  
  하이버네이트의 경우 경고 로그를 남기고 데이터 전부 불러온뒤에 메모리에서 페이징한다.  

### 일대다 페치 조인 페이징 해결방법
* **일대다를 다대일로 뒤집어서 페이징을 수행한다.**
* **배치사이즈 기능을 이용한다.**  
  아래의 코드를 실행하면 페치 조인이 없기 때문에 N+1문제가 발생된다.  
  ```java
  em.createQuery("select m FROM Member m", Member.class)
    .setFirstResult(0)
    .setMaxResults(5)
    .getResultList();
  ```
  하지만 이때 배치사이즈를 다음과 같이 설정하면
  ```
  [글로벌]
  hibernate.default_batch_fetch_size = 보통 1000이하의 값
  [지정]
  @BatchSize(size = 100)
  @OneToMany(mappedBy = "member")
  private List<Order> orders = new ArrayList<>();
  ```
  설정한 배치 사이즈 만큼 Order를 한번에 불러온다.
  ```sql
  select  o.*, 
  from    orders o 
  where   orders.member_id in ( batch size 이내 )
  ```
* **DTO를 사용한다**

## 정리
* 모든 것을 페치 조인으로 해결할 수 는 없다.
* 페치 조인은 객체 그래프를 유지할 때 사용하면 효과적
* **여러 테이블을 조인해서 엔티티가 가진 모양이 아닌 전혀 다른 결과를 내야 하면, 페치 조인 보다는 일반 조인을 사용하고 필요한 데이터들만 조회해서 DTO로 반환하는 것이 효과적**

