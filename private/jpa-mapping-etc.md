# JPA 고급 매핑
## 상속관계 매핑
관계형 데이터베이스에는 상속 관계가 없다.  
대신 이와 유사한 슈퍼타입 서브타입 관계라는게 존재한다.  
JPA에서는 객체의 상속과 데이터베이스의 슈퍼타입 서브타입 관계를 매핑해주는 '상속관계 매핑'을 지원한다.  
  
슈퍼타입 서브타입 논리 모델은 크게 3가지 전략으로 물리 모델로 구현될 수 있다.
### JOINED 전략
서브타입 테이블의 PK로 슈퍼타입의 PK를 사용하는 전략이다.  
```java
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
@DiscriminatorColumn  //서브타입을 구분해주는 컬럼을 생성하는 에노테이션 (name 속성의 기본값은 DTYPE)
public abstract class Item 
```
> **장점**  
테이블 정규화  
외래 키 참조 무결성 제약조건 활용가능  
저장공간 효율화  
**단점**  
조회시 조인을 많이 사용, 성능 저하  
조회 쿼리가 복잡함  
데이터 저장시 INSERT SQL 2번 호출  

### SINGLE_TABLE 전략
슈퍼타입 서브타입을 전부 하나의 테이블로 통합해서 사용하는 전략이다.  
```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn  //SINGLE_TABLE전략에서는 DTYPE 컬럼이 필수이므로 해당 코드가 없어도 자동으로 들어간다.
public abstract class Item {
}

@Entity
@DiscriminatorValue("B")  //DTYPE 컬럼에 들어갈 값을 설정한다. (기본값 : 엔티티의 이름)
public class Book extends Item {

    private String author;
    private String isbn;
}
```
> **장점**  
조인이 필요 없으므로 일반적으로 조회 성능이 빠름  
조회 쿼리가 단순함  
저장공간 효율화  
**단점**  
자식 엔티티가 매핑한 컬럼은 모두 null 허용  
단일테이블에모든것을저장하므로테이블이커질 수 있다.  
상황에 따라서 조회 성능이 오히려 느려질 수 있다.

### TABLE_PER_CLASS 전략
슈퍼타입 없이 서브타입 별로 테이블을 생성하는 전략이다.  
```java
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
public abstract class Item  //TABLE_PER_CLASS전략에서는 abstract가 아닐경우 Item클래스도 엔티티로 생성이 되기 때문에 직접 사용할 일이없다면 abstract를 하자
```
> 쓰지않는것이 좋다.  

## @MappedSuperclass
테이블과는 관계 없고, 단순히 엔티티가 공통으로 사용하는 매핑정보를 모으는 역할을 한다.  
@MappedSuperclass가 붙은 클래스는 엔티티가 아니고 테이블과 매핑되지도 않는다.  
```java
@MappedSuperclass
public abstract class BaseEntity { //직접 생성해서 사용할 일이 없으므로 추상클래스 권장

    private String createdBy;
    private LocalDateTime createdDate;
    private String lastModifiedBy;
    private LocalDateTime lastModifiedDate;
}
```
```java
@Entity
public class Member extends BaseEntity {
}
```
> **참고**  
엔티티는 @Entity나 @MappedSuperclass로 지정한 클래스만 상속 받을 수 있다.  


