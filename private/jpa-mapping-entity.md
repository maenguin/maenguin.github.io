
# JPA 엔티티 매핑  
JPA는 ORM으로서 객체와 테이블 매핑을 위해 다양한 에노테이션을 지원한다.  

## 객체와 테이블 매핑  
  
### @Entity  
클래스에 `@Entity`가 붙으면 테이블과 매핑이되며 JPA에서 관리되어진다.  
`@Entity`가 붙은 클래스를 `엔티티`라 한다.  
> **주의**  
JPA내부적으로 리플랙션 기능을 이용하기 위해 **기본 생성자**가 필수로 들어가야 된다.(public or protected)  
final 클래스, enum, interface, inner 클래스에는 사용 불가.  
저장할 필드에 final을 사용하면 안된다.  

#### @Entity 속성
* **name**  
  JPA에서 사용할 엔티티 이름을 지정하는 속성(테이블 이름이 아니다)  
  기본값은 클래스 이름이며 다른 패키지에 같은 클래스 이름이 없는 이상 가급적 기본값을 사용한다.  
    
### @Table  
`@Table`은 엔티티와 매핑할 테이블을 명시적으로 지정할때 사용한다.  
#### 속성
* **name**  
  매핑할 실제 테이블의 이름이 들어가며 기본값은 엔티티 이름  
* **catalog**  
  DB의 catalog를 매핑한다.  
* **schema**  
  DB의 schema를 매핑한다.
* **uniqueConstraints**  
  DDL 생성 시에 유니크 제약 조건을 생성  
  ```java
  @Table(uniqueConstraints = {@UniqueConstraint( name = "NAME_AGE_UNIQUE",
                                                 columnNames = {"NAME", "AGE"} )})
  ```
> name속성의 경우 스프링부트에선 카멜케이스로 적으면 자동으로 `_`로 변환되어 매핑된다.(ex orderHistory -> order_history)  
따로 컨벤션을 지정할 수 도 있다.

> DDL 생성 기능은 DDL을 자동 생성할 때만 사용되고 JPA 실행 로직에는 영향을 주지 않는다.  

## 필드와 컬럼 매핑
### @Column
```java
@Column(
    name = "name", //필드와 매핑할 테이블의 컬럼 이름 (기본값 : 객체의 필드 이름)
    insertable = true,    //등록 가능 여부 (기본값 : true)
    updatable = true,    //변경 가능 여부 (기본값 : true)
    nullable = true,     //(DDL용) false로 설정하면 DDL 생성 시에 not null 제약조건이 붙는다. (기본값 : true)
    uinque = false,       //(DDL용) true로 설정하면 DDL 생성 시에 unique 제약 조건이 추가됨 (기본값 : false)
    lenght = 255,         //(DDL용) 문자 길이 제약조건, String 타입에만 사용한다. (기본값 : 255)
    columnDefinition = "varchar(100) default 'EMPTY'", //데이터베이스 컬럼 정보를 직접 줄 수 있다.
    
    precision = 19,     //BigDecimal or BigInterger 타입에서 사용한다. 정밀한 소수를 다루어야 할 때 사용.
    sacale = 2,         //precision은 소수점을 포함한 전체 자릿수를, scale은 소수의 자릿수를 의미한다.
    )
private String username;
```
> @Column으로 유니크 제약조건을 걸게 되면 제약조건 이름을 사용자가 지정할 수 없다.  
그러므로 왠만하면 @Table의 유니크 제약조건을 사용하도록 하자.  

### @Enumerated
자바 enum 타입을 매핑할 때 사용한다.  
* EnumType.ORDINAL : enum **순서**를 데이터베이스에 저장 (기본값)
* EnumType.STRING : enum **이름**을 데이터베이스에 저장
```java
@Enumerated(EnumType.STRING)
private RoleType roleType;
```
> **주의**  
ORDINAL를 사용하게 되면 나중에 추가된 enum 요소 때문에 순서가 꼬일 수 있다.  
무조건 STRING을 사용하자.  

### @Temporal
날짜 타입을 매핑할 때 사용한다.  
자바 8이상 부터는 LocalDate지원하기 때문에 사용하지 않아도 된다.  

### @Lob
데이터베이스 BLOB, CLOB 타입과 매핑한다.  
필드 타입이 문자면 CLOB, 그 이외에는 BLOB으로 자동 매핑된다.  

### @Transient
데이터베이스와 매핑하지 않고 메모리상에서만 임시로 사용할 때 사용한다.  
JPA는 기본적으로 자동 매핑이기 때문에 원하지 않는 매핑 필드가 있으면 사용한다.  


## 기본키 매핑
기본키 매핑을 위해서 @Id와 @GeneratedValue 두개의 에노테이션 존재한다.  
  
### @Id
원하는 필드에 붙이면 PK로 설정된다.  
이후에 `@GerneratedValue`를 사용하지 않으면 엔티티 저장시 직접 id를 할당해야한다.  

### @GeneratedValue
PK 생성 전략을 설정할 수 있다.  

* **AUTO**  
  기본값, 데이터베이스 방언에 따라 밑 3가지 전략중 하나가 자동 선택된다.  

* **IDENTITY**  
  기본 키 생성을 데이터베이스에 위임하는 전략  
  주로 MySQL, PostgreSQL, SQL Server, DB2에서 사용 (ex: MySQL의 AUTO_INCREMENT)  
  > AUTO_INCEREMENT는 DB에 INSERT SQL을 실행한 이후에야 ID 값을 알 수 있다.  
  여기서 JPA는 ID값이 있어야 영속할 수 있기 때문에 IDENTITY 전략은 다른 전략과는 예외적으로  
  em.persist() 시점에 즉시 INSERT SQL을 실행하고 ID를 얻어온다.  

* **SEQUENCE**  
  DB 시퀀스 오브젝트를 사용하는 전략  
  오라클, PostgreSQL, DB2, H2에서 사용  
  `@SequenceGenerator`를 통해 생성한 시퀀스를 매핑할 수 있다.  
  ```java
  @Entity
  @SequenceGenerator(
          name = "MEMBER_SEQ_GENERATOR" //식별자 생성기 이름 (필수)
          sequenceName = "MEMBER_SEQ",  //매핑할 데이터베이스 시퀀스 이름 (기본값 : hibernate_sequence)
          initialValue = 1,             //DDL 생성 시에만 사용됨, 초기값을 지정한다. (기본값 : 1)
          allocationSize = 1            //시퀀스 한 번 호출에 증가하는 수 (기본값 : 50)
          ) 
  public class Member {
      @Id
      @GeneratedValue(strategy = GenerationType.SEQUENCE,
                      generator = "MEMBER_SEQ_GENERATOR")
      private Long id;
  }
  ```
  > @SequenceGenerator에서 allocationSize의 기본값은 50이다.  
  **DB 시퀀스 값이 하나씩 증가하도록 설정되어 있으면 반드시 1로 설정해야한다.**  

  > SEQUENCE 전략도 엔티티 영속을 위해 어쩔 수 없이 DB에 접근해 시퀀스 값을 가져와야한다.  
  IDENTITY 전략과는 다르게 em.persist()시 시퀀스 값만 call하고 INSERT SQL을 실행하지 않기 때문에 버퍼링을 할 수 있지만 성능 이슈가 있을 수 있다.   
  이때 allocationSize를 설정해서 성능 최적화를 할 수 있다.  
  기본 값이 50인데 이렇게 설정하면 시퀀스값 50개를 미리 받아오고 이후부터는 call없이 메모리 상에서 시퀀스를 증가시킨다.  
  웹서버가 여러대인경우 서버1은 1\~50 서버2는 51\~100을 미리받는식으로 동작하기 때문에 동시성 문제도 걱정없다.  

* **TABLE**  
  키 생성 전용 테이블을 하나 만들어서 데이터베이스 시퀀스를 흉내내는 전략  
  모든 데이터베이스에 적용 가능 하지만 성능 때문에 잘 사용하지 않는다.  
  `@TableGenerator`로 생성한 테이블을 매핑할 수 있다.  
  ```ddl
  create table MY_SEQUENCES (
      sequence_name varchar(255) not null,
      next_val bigint,
      primary key (sequence_name)
  }
  ```
  ```java
  @Entity
  @TableGenerator(
          name = "MEMBER_SEQ_GENERATOR"    //식별자 생성기 이름 (필수)
          table = "MY_SEQUENCE",           //키 생성 테이블명 (기본값 : hibernate_sequences)
          pkColumnValue = "MEMBER_SEQ",    //키로 사용할 값 이름 (기본값 : 엔티티 이름)
          pkColumnValue = "sequence_name", //시퀀스 컬럼명 (기본값 : sequence_name)
          valueColumnNa = "next_val",      //시퀀스 값 컬럼명 (기본값 : next_val)
          initialValue = 0,                //초기값 (기본값 : 0)
          allocationSize = 1               //시퀀스 한 번 호출에 증가하는 수 (기본값 : 50)
          ) 
  public class Member {
      @Id
      @GeneratedValue(strategy = GenerationType.TABLE,
                      generator = "MEMBER_SEQ_GENERATOR")
      private Long id;
  }
  ```
  ```table
  MY_SEQUENCE 테이블에
  sequence_name next_val
  MEMBER_SEQ    1
  이런식으로 저장됨
  ```  
  
  
