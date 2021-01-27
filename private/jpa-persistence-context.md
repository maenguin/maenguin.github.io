# JPA 영속성 컨텍스트
엔티티를 영구 저장하는 환경  
JPA는 영속성 컨텍스트라는 환경에서 엔티티를 관리한다.  
엔티티 매니저를 통해 접근  
> J2SE환경에서는 엔티티 매니저와 영속성 컨텍스트가 1:1 관계이지만  
J2EE, 스프링 프레임워크 같은 컨테이너 환경에서는 엔티티 매니저와 영속성 컨텍스트가 N:1의 관계이다.

## 엔티티의 상태
영속성 컨텍스트를 기준으로 다음과 같이 엔티티의 상태를 나눌 수 있다.  

* **비영속** (new/transient)  
  영속성 컨텍스트와 전혀 관계가 없는 새로운 상태
  ```java
  Member member = new Member(1L, "회원1");
  ```
* **영속** (managed)  
  영속성 컨텍스트에 관리되는 상태  
  ```java
  Member member = new Member(1L, "회원1");
  em.persist(member);
  ```
* **준영속** (detached)  
  영속성 컨텍스트에 저장되었다가 분리된 상태
  ```java
  em.detach(member);
  em.clear(); //영속성 컨텍스트 내 엔티티를 모두 준영속으로 전환함.
  em.close();
  ```  
  준영속 상태가 되면 영속성 컨텍스트가 제공하는 기능을 사용하지 못한다.  

* **삭제** (remved)  
  삭제된 상태
  ```java
  em.remove(member);
  ```

## 영속성 컨텍스트의 이점  

* **1차 캐시**  
  영속성 컨텍스트는 `1차 캐시`라는 저장공간을 가지며 이는 말그대로 캐시 기능을 수행한다.  
  특정 엔티티를 찾을때, 그 엔티티가 이미 1차 캐시에 올라와 있는 상황이라면 DB를 조회하지 않고 1차 캐시에 있는 엔티티를 반환하게 된다.  
  em.persist()를 통해 엔티티를 영속화하면 자동으로 1차 캐시에 저장되며 DB에서 가져온 엔티티가 1차 캐시에 없어도 자동으로 저장된다.  
  ```java
  Member member = new Member(1L, "회원1");

  //1차 캐시에 저장됨
  em.persist(member);
  //1차 캐시에서 조회, 1차 캐시에 있으니 DB를 거치지 않음
  em.find(Member.class, 1L);
  //1차 캐시에 없음, DB에서 조회 -> 1차 캐시에 저장 -> 반환
  em.find(Member.class, 2L);
  ```  


* **영속 엔티티의 동일성 보장**  
  기존의 메커니즘에서는 같은 키를 가진 엔티티를 두번 가져오더라도 객체의 관점에서 보았을때 다른 엔티티로 식별된다.  
  하지만 JPA에선 영속성 컨텍스의 1차 캐시 기능 덕에 영속 엔티티의 동일성을 보장한다.  
  ```java
  Member a = em.find(Member.class, "member1");
  Member b = em.find(Member.class, "member1");
  System.out.println(a == b); //true
  ```
  > 1차 캐시로 반복 가능한 읽기(Repeatable Read)등급의 트랜잭션 격리 수준을 데이터베이스가 아닌 애플리케이션 차원에서 제공  


* **트랜잭션을 지원하는 쓰기 지연 (transactional write-behind)**  
  트랜잭션이 시작된 후 데이터를 저장, 변경, 삭제하는 코드를 호출해도 해당 SQL이 바로 DB에 보내지지 않는다.   
  대신 생성된 SQL이 영속성 컨텍스트의 `쓰기 지연 SQL 저장소`에 차곡차곡 쌓이고 트랜잭션이 커밋되는 순간에 DB로 보내게 된다.  
  ```java
  transaction.begin();

  em.persist(memberA);
  em.remove(memberB);
  memberC.changeName("memberD");
  //여기까지 SQL를 DB에 보내지 않음

  //커밋하는 순산 DB에 쿼리를 보냄
  transaction.commit();
  ```  

* **변경 감지 (dirty checking)**  
  영속성 컨텍스트는 엔티티를 영속할때 1차 캐시에 해당 엔티티의 `스냅샷`을 저장해 놓는다.  
  이 스냅샷과 현재 엔티티를 비교해 변경사항을 감지하고 만약 엔티티가 변경되었다면 SQL을 생성하여 쓰기 지연 SQL 저장소에 저장한다.
  ```java
  transaction.begin();

  Member memberA = em.find(Member.class, "memberA");
  memberA.changeName("memberB");
  //memberA.persist(memberA); 를 안해도 자동으로 변경 감지 되어 SQL이 생성된다!
  transaction.commit();
  ```  

* **지연 로딩 (lazy loading)**
  

## 플러시  
영속성 컨텍스트의 변경내용을 데이터베이스에 반영하는것  
플러시가 호출되면 변경 감지가 실행되어 수정된 엔티티에 대한 SQL이 쓰기 지연 SQL 저장소에 등록되고 등록된 쿼리들이 DB에 전송된다.  
transaction.commit()시 자동으로 호출되며 em.flush()를 통해 직접 호출 할 수 도 있다.  

> JPQL 쿼리를 실행할때도 플러시가 자동 호출된다. (동시성 보장을 위함)  
쿼리 호출시에 플러시를 방지하고 싶다면 em.setFlushMode(FlushModeType.COMMIT)로 설정하면된다.  
  
  
## Merge
**JPA에서 수정을 하는법**  
다음과 같이 Item 엔티티의 update를 처리하는 서비스가 있다고 해보자.  
파라미터로 넘어온 Item 엔티티는 영속인지 준영속인지 알 수 없다고 가정하자.
```java
@Transactional
void update(Item item) {
    // ?
}
```
JPA에서는 어떤 방식으로 엔티티를 수정해야될까?

### 변경 감지
위에서 알아봤던 변경 감지 기능을 사용하여 수정할 수 있다.  
이 경우에는 변경 감지 기능을 사용하기 위해 **엔티티를 영속 상태**로 만들어 줘야 한다.  
```java
@Transactional
void update(Item item) {
    Item findItem = em.find(Item.class, item.getId());
    findItem.changePrice(item.getprice());
}
```
이 경우 트랜잭션 커밋 시점에 변경 감지가 동작해서 데이터베이스에 update SQL이 실행된다.  

### 병합
병합은 준영속 상태의 엔티티를 영속 상태로 변경할 때 사용한는 기능이다.
```java
@Transactional
void update(Item item) {
    Item mergeItem = em.merge(item);
}
```
1. merge() 실행하면
2. 파라미터로 넘어온 준영속 엔티티의 식별자 값으로 1차 캐시에서 엔티티를 조회한다.
  2.1 만약 1차 캐시에 엔티티가 없으면 데이터베이스에서 엔티티를 조회하고, 1차 캐시에 저장한다.
3. 조회한 영속 엔티티의 값을 준영속 엔티티의 값으로 모두 교체한다.(병합한다.)
4. 영속 엔티티를 반환한다.
5. 트랜잭션 커밋 시점에 변경 감지 기능이 동작해서 데이터베이스에 update SQL이 실행된다.  

### 병합의 문제점
변경 감지 기능을 사용하면 원하는 속성만 선택해서 변경할 수 있지만  
병합을 사용하면 **모든 속성이 변경되어 병합시 값이 없으면 `null`로 업데이트 될 위험이 있다**  

### 병합의 문제 해결 방법
**엔티티를 변경할 때는 항상 변경 감지를 사용**하는것이 좋다.
* 컨트롤러에서 어설프게 엔티티를 생성하지 않는다.
* 트랜잭션이 있는 서비스 계층에 `식별자`와 `변경할 데이터`를 명확하게 전달한다. (파라미터 or dto)
* 트랜잭션이 있는 서비스 계층에서 영속 상태의 엔티티를 조회하고, 엔티티의 데이터를 직접 변경한다.  





