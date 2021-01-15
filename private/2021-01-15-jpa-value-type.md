# 값 타입

## JPA의 데이터 타입 분류
* 엔티티 타입  
  * @Enity로 정의하는 객체
  * 데이터가 변해도 식별자로 지속해서 추적 가능
* 값 타입
  * int, Integer, String 처럼 단순히 값으로 사용하는 자바 기본 타입이나 객체
  * 식별자가 없고 값만 있으므로 변경시 추적 불가
  
## 값 타입 분류
* 기본값 타입
  * 자바 기본 타입(int, double)
  * 래퍼 클래스(Integer, Long)
  * String
* 임베디드 타입(embedded type, 복합 값 타입)
* 컬렉션 값 타입(collection value type)

## 기본값 타입
* 생명주기를 엔티티에 의존 (회원을 삭제하면 이름, 나이 필드도 함께 삭제)
> 자바 기본 타입은 공유되지 않는다. 대입시 값이 복사된다.  
래퍼 클래스는 참조 타입이기 때문에 공유는 할 수 있지만 불변이기 때문에 변경이 불가능하다.  

## 임베디드 타입
새로운 값 타입을 직접 정의할 수 있다. JPA에서는 이를 `임베디드 타입`이라고 한다.  
주로 기본값 타입을 모아서 만들기 때문에 `복합 값 타입`이라고도 한다.  
```java
@Entity
public class Member extends BaseEntity {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    private String name;
    
    private String city;
    private String street;
    private String zipCode;
    
    //..
}
```
city, street, zipcode를 묶어서 Address라는 값 타입을 만들어보자
```java
@Embeddable //값 타입을 정의하는 곳에 표시한다.
public class Address {
    //임베디드 타입은 기본 생성자가 필수로 있어야한다.
    private String city;
    private String street;
    private String zipcode;
}
```
```java
@Entity
public class Member extends BaseEntity {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    private String name;
    
    @Embedded //값 타입을 사용하는 곳에 표시한다.
    private Address address;
    
    //..
}
```
> @Embeddable과 @Embedded 중 하나만 표시 해도 되지만  
둘다 명시하는걸 권장  

> 임베디드 타입의 값이 null이면 매핑한 컬럼 값도 모두 null이 된다.  

### 임베디드 타입의 장점
* 재사용  
* 높은 응집도  
* Address.Method() 처럼 해당 값 타입만 사용하는 의미 있는 메소드를 만들 수 있음  
* 임베디드 타입을 포함한 모든 값 타입은, 값 타입을 소유한 엔티티에 생명주기를 의존함  

### 임베디드 타입과 테이블 매핑
* 임베디드 타입은 엔티티의 값일 뿐이다.
* 임베디드 타입을 사용하기 **전과 후에 매핑하는 테이블은 같다.**
* 객체와 테이블을 아주 세밀하게(find-grained) 매핑하는 것이 가능  
* 잘 설계한 ORM 애플리케이션은 매핑한 테이블의 수보다 클래스의 수가 더많음  

### @AttributeOverride: 속성 재정의
한 엔티티에서 같은 값 타입을 사용하면 컬럼 명이 중복된다.  
`@AttributeOverrides`와 `@AttributeOverride`를 사용해서 컬럼명 속성을 재정의 할 수 있다.  
```java
@Entity
public class Member extends BaseEntity {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    private String name;

    @Embedded
    private Address homeAddress;

    @Embedded
    @AttributeOverrides({
            @AttributeOverride(name="city", column=@Column("WORK_CITY")),
            @AttributeOverride(name="street", column=@Column("WORK_STREET")),
            @AttributeOverride(name="zipcode", column=@Column("WORK_ZIPCODE"))
     })
    private Address workAddress;
    
    //..
}
```

## 값 타입과 불변 객체
앞서 보았듯이 기본 값 타입은 공유가 불가능 하고 가능한 경우라도 불변이기 때문에 안전하다.  
하지만 임베디드 타입은 값 타입이지만 객체이므로 공유가 가능하면서 변경이 가능하다.  
```java
Address address = new Address("city1", "street1", "zipcode1");
member1.setAddress(address);
member2.setAddress(address);

//member1의 city만 바꾸려고 했지만 member2의 city값도 바뀌어 버린다.
member1.getAddress().setCity("city2");
```
그렇기 때문에 값을 복사해서 사용해야 되지만 코드적으로 실수할 가능성이 있다.  
그러므로 자바의 String처럼 **불변 객체(immutable object)로 설계해야한다.**
```java
Address address = new Address("city1", "street1", "zipcode1");
member1.setAddress(address);
member2.setAddress(address);

//불변객체를 만드는 방법은 여러가지가 있으나 여기서는 setter를 없애고 생성자에서만 초기화 할 수 있도록 했다.
//member1.getAddress().setCity("city2");  
member1.setAddress(new Address("city2", address.getStree(), address.getZipcode()));
```
