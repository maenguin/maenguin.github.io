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
            @AttributeOverride(name="city", column=@Column(name = "WORK_CITY")),
            @AttributeOverride(name="street", column=@Column(name = "WORK_STREET")),
            @AttributeOverride(name="zipcode", column=@Column(name = "WORK_ZIPCODE"))
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

## 값 타입 비교
값 타입은 인스턴스가 달라도 그 안에 값이 같으면 같은 것으로 봐야한다.  
`== 의 경우 동일성 비교`로서 인스턴스의 참조 값을 비교한다.  
`equals()의 경우 동등성 비교`로서 인스턴스의 값을 비교한다.  
그러므로 값 타입은 **equals()를 사용해서 동등성 비교를 해야한다.**  
  
다음과 같이 equals() 메소드를 적절하게 재정의 해서 사용한다.  
```java
@Embeddable
@Getter
public class Address {

    private String city;
    private String street;
    private String zipcode;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Address address = (Address) o;
        return Objects.equals(city, address.city) &&
                Objects.equals(street, address.street) &&
                Objects.equals(zipcode, address.zipcode);
    }

    @Override
    public int hashCode() {
        return Objects.hash(city, street, zipcode);
    }
}
```

## 값 타입 컬렉션
값 타입이 한 두개가 아니라 정말 여러개 일때는 `값 타입 컬렉션`이라는것을 사용할 수 있다.  
데이터베이스는 컬렉션을 같은 테이블에 저장할 수 없기 때문에 별도의 테이블이 필요하다.  
  
`@ElementCollection`과 `@CollectionTable`을 사용해서 만들 수 있다.  
```java
@ElementCollection
@CollectionTable(name = "FAVORITE_FOOD", joinColumns = @JoinColumn(name = "MEMBER_ID"))
@Column(name = "FOOD_NAME") //컬렉션이 단일 기본 값타입 하나 이므로 예외적으로 @Column사용가능
private Set<String> favoriteFood = new HashSet<>();

@ElementCollection
@CollectionTable(name = "ADDRESS", joinColumns = @JoinColumn(name = "MEMBER_ID"))
private List<Address> addressHistory = new ArrayList<>();
```

### 값 타입 컬렉션의 사용
값 타입 컬렉션은 `영속성 전이 + 고아 객체 제거`를 설정 한것 처럼 동작 한다.  
* 저장  
  ```java
  Member member = new Member();
  
  member.getFavoriteFoods().add("치킨");
  member.getFavoriteFoods().add("족발");
  member.getFavoriteFoods().add("피자");
  
  member.getAddressHistory().add(new Address("city1", street1", "zipcode1"));
  member.getAddressHistory().add(new Address("city2", street2", "zipcode2"));
  
  //member만 persist해도 FAVORITE_FOOD 테이블과 ADDRESS 테이블에 INSET가 나간다.
  em.perist(member);
  ```
* 조회
  ```java
  //..
  
  //값 타입 컬렉션은 기본이 지연로딩이기 때문에 Member 데이터만 가져온다.
  em.find(Member.class, member.getId());
  ```
* 수정
  ```java
  //..
  Member findMember = em.find(Member.class, member.getId());
  findMember.getFavoriteFoods().remove("치킨");
  findMember.getFavoriteFoods().add("한식");
  ```
  ```java
  //..
  //remove는 내부적으로 hashcode를 이용하기 때문에 아래 코드로도 동작한다.
  findMember.getAddressHistory().remove(new Address("city1", street1", "zipcode1"));
  findMember.getAddressHistory().add(new Address("newcity1", street1", "zipcode1"));
  //잘 되긴 하는데 값을 모두 제거하고 컬렉션에 있는 값을 다시 저장한다!
  ```
### 값 타입 컬렉션의 제약사항
* 값 타입은 엔티티와 다르게 식별자 개념이 없다.  
* 값은 변경하면 추적이 어렵다
* 값 타입 컬렉션에 변경 사항이 발생하면, 주인 엔티티와 연관된 모든 데이터를 삭제하고,  
  값 타입 컬렉션에 있는 현재 값을 모두 다시 저장한다.  
* 값 타입 컬렉션을 매핑하는 테이블은 모든 컬럼을 묶어서 기본키를 구해성해야 한다 => null x, 중복 저장 x  
### 값 타입 컬렉션 대안
**값 타입 컬렉션 대신 일대다 관계를 고려**  
1. 일대다 관계를 위한 엔티티를 만들고 여기에 값 타입을 사용  
2. 영속성 전이와 고아 객체 제거를 사용해서 값 타입 컬렉션 처럼 사용  
```java
@Entity
public class AddressEntity {

    @Id
    @GeneratedValue
    private Long id;

    private Address address;
}
```
```java
@OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
@JoinColumn(name = "MEMBER_ID")
private List<AddressEntity> addressHistory = new ArrayList<>();
```


  
  
  
  
  
  
