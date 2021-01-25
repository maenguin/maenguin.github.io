# JPA 연관관계 매핑
객체와 테이블의 연관관계는 서로 차이가 있다.  
Member와 Order가 1:N 관계라고 가정해보자  
기존 방식대로 **객체를 테이블에 맞추어 모델링**을 한다면 아래와 같을것이다.    
```java
@Entity
public class Member {
    @Id @GeneratedValue
    @Column(name = "member_id")
    private Long id;
    private String name;
    //...
}
```
```java
@Entity
public class Order {
    @Id @GeneratedValue
    @Column(name = "order_id")
    private Long id;
    @Column(name = "member_id")
    private Long memberId;
    //...
}
```
테이블에 맞췄기 때문에 **Order 클래스는 memberId를 외래키**로 가지고 있는걸 볼 수 있다.  
관계형 모델 관점에서는 문제가 없지만 객체 지향 관점에서 위 클래스는 객체지향적이라고 할 수 없다.  
Order 클래스는 memberId가 아니라 Member를 참조로 가지고 있어야 한다.  
JPA에서는 이러한 패러다임 불일치를 해결하기 위해 **객체의 참조와 테이블의 외래 키를 매핑**하는 연관관계 매핑을 지원한다.  
  
`@XToX`과 `@JoinColumn` 에노테이션을 이용해 매핑이 가능하다.  
```java
@Entity
public class Order {

    @Id @GeneratedValue
    @Column(name = "order_id")
    private Long id;
    
    @ManyToOne
    @JoinColumn(name = "member_id")
    private Member member;
    
    //...
}
```


## 방향 설정
JPA에서는 `단방향`과 `양방향`이라는 개념이 등장한다.  
객체 모델링을 할때 연관관계가 있는 객체 한쪽에만 연관관계 매핑을 하면 단방향, 양쪽에 하면 양방향이다.  
단방향으로 연관관계 매핑을 하면 한 객체에서만 다른 객체를 참조할 수 있다.  
양방향으로 연관관계 매핑을 하면 양 객체에서 각각 다른 객체를 참조 할 수 있다.  
### 단방향 연관관계
위 코드 처럼 한쪽에만 연관관계 매핑을 해주면 단방향 연관관계가 된다.
### 양방향 연관관계
한쪽에 단방향 연관관계 설정 후 반대쪽 객체를 기준으로 또 단방향 연관관계를 매핑해주면 양방향 연관관계가 된다.  
#### 연관관계 주인
양방향 연관관계인 경우 **연관관계의 주인**을 설정해야한다.  
왜냐하면 JPA는 연관관계가 매핑된 필드의 변경을 감지해서 DB를 업데이트 하는데, 양방향 연관관계는 양쪽에 연관관계가 매핑되어 있기때문에  
어느것을 기준으로 동작해야하는지 알 수 없게 된다. 그러므로 변경의 주체를 명시해줘야하는데 그게 연관관게의 주인 설정이다.  
연관관계의 주인이 아닌 필드는 읽기만 가능해진다.  
`mappedBy`로 주인이 아님을 명시할 수 있다.  

```java
@Entity
public class Member {

    @Id @GeneratedValue
    @Column(name = "member_id")
    private Long id;
    
    private String name;
    
    @OneToMany(mappedBy = "member") //Order class에 있는 member필드에 의해 매핑되었다라는 뜻이다.
    private List<Order> orders = new ArrayList<>();
    //...
}
```  
> 비즈니스 룰에 얽매이지 않고 **외래 키가 있는 곳을 주인**으로 정하는게 좋다!!  

> **주의1**  
> 양방향 연관관계 일때 주인에 값을 입력하지 않는 실수를 할 수 있다.  
>  ```java
>  Member member = new Member("mem1");
>  em.persist(member);
>
>  Order order = new Order();
>  //order.setMember(member);
>  member.getOrders().add(order); //역방향만 연관관계 설정
>  em.persist(order);
>  ```
>  그러면 DB에 외래키가 null이 들어가므로 주의해야한다.  
>  **순수 객체 상태를 고려해서 항상 양쪽에 값을 설정하는것이 좋다.**  
>  연관관계 편의 메소드 사용하자  

> **주의2**  
양방향 연관관계는 무한루프에 빠질 위험이 있다.  
toString(), lombok, json 생성 라이브러리  

> **단방향 매핑만으로도 이미 연관관계 매핑은 완료된다.**  
그러므로 양방향은 역방향 탐색이 필요할 때 추가하도록 한다.  

***
