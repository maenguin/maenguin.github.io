# 스프링 데이터 JPA 확장 기능
* [사용자 정의 리포지토리 구현](#사용자-정의-리포지토리-구현)
* [Auditing](#Auditing)
* [Web 확장 - 도메인 클래스 컨버터](#Web-확장---도메인-클래스-컨버터)
* [Web 확장 - 페이징과 정렬](#Web-확장---페이징과-정렬)
* [스프링 데이터 JPA 구현체 분석](#스프링-데이터-JPA-구현체-분석)
* [새로운 엔티티를 구별하는 방법](#새로운-엔티티를-구별하는-방법)
* 그외
  * [Projections](#Projections)
  * [네이티브 쿼리](#네이티브-쿼리)
  * Specifications
  * Query By Example  


## 사용자 정의 리포지토리 구현
스프링 데이터 JPA 리포지토리는 인터페이스만 정의하고 구현체는 스프링이 자동 생성한다.  
**메서드를 직접 구현하고 싶다면 어떻게 해야될까?**  
인터페이스를 상속받아서 직접 구현하려면 구현해야 하는 기능이 너무 많다.  
이때 다음과 같이 사용자 정의 리포지토리를 구현할 수 있다.  

1. 먼저 사용자 정의 인터페이스를 만든다.  
    ```java
    public interface MemberRepositoryCustom {
      List<Member> findMemberCustom();
    }
    ```
2. 사용자 정의 인터페이스를 상속받은 구현 클래스를 만든다.
    ```java
    @RequiredArgsConstructor
    public class MemberRepositoryImpl implements MemberRepositoryCustom {

      private final EntityManager em;

      @Override
      public List<Member> findMemberCustom() {
          return em.createQuery("select m from Member m")
                  .getResultList();
      }
    }
    ```  
3. 사용자 정의 인터페이스를 기존에 사용하던 JPA 리포지토리 인터페이스에 상속한다.  
    ```java
    public interface MemberRepository extends JpaRepository<Member, Long> , MemberRepositoryCustom {
        ...
    }
    ```
4. 사용자 정의 메서드를 호출해서 사용한다.  
    ```java
    memberRepository.findMemberCustom();
    ```  
  
* 사용자 정의 구현 클래스는 **리포지토리 인터페스 이름 + `Impl`** 로 명명해야 스프링 데이터 JPA가 인식해서 스프링 빈으로 등록한다.  
* 명명규칙을 변경하고 싶으면 `@EnableJpaRepositories에 respotirogyImplementationPostfix 속성`을 이용하면 된다.  
* 스프링 데이터 2.x 부터는 **사용자 정의 인터페이스 명 + `Impl`** 방식도 지원한다. (이 방법을 더 권장)  
* 실무에서는 주로 QueryDSL이나 SpringJdbcTemplate를 함께 사용할 때 사용자 정의 리포지토리 기능을 자주 사용한다.  

***

## Auditing
엔티티를 생성, 변경할 때 변경한 사람과 시간을 추적하고 싶다면?

### 순수 JPA Auditing
```java
@Getter
@MappedSuperclass
public class JpaBaseEntity {

    @Column(updatable = false)
    private LocalDateTime createdDate;
    private LocalDateTime updatedDate;

    @PrePersist
    public void prePersist() {
        LocalDateTime now = LocalDateTime.now();
        createdDate = now;
        updatedDate = now;
    }

    @PreUpdate
    public void preUpdate() {
        updatedDate = LocalDateTime.now();
    }
}
```
* JPA 이벤트를 이용해서 직접 생성, 변경 날짜를 입력해준다.  

### 스프링 데이터 JPA 사용
```java
@EntityListeners(AuditingEntityListener.class)
@Getter
@MappedSuperclass
public class BaseEntity {

    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createdDate;

    @LastModifiedDate
    private LocalDateTime lastModifiedDate;

    @CreatedBy
    @Column(updatable = false)
    private String createdBy;

    @LastModifiedBy
    private String lastModifiedBy;
}
```
```java
@EnableJpaAuditing
@SpringBootApplication
public class DataJpaApplication
```
```java
@Bean
public AuditorAware<String> auditorProvider() {
 return () -> Optional.of(UUID.randomUUID().toString());
}
```
* 엔티티에 `@EntityListeners(AuditingEntityListener.class)` 적용  
* 스프링 부트 설정 클래스에 `@EnableJpaAuditing` 적용
* 등록자, 수정자를 처리하고 싶으면 `AuditorAware`를 스프링 빈으로 등록하면 된다.  
  * 실무에서는 세션 정보나, 스프링 시큐리티 로그인 정보에서 ID를 받는다.  
* 저장시점에 저장데이터만 입력하고 싶으면 `@EnableJpaAuditing(modifyOnCreate = false)`옵션을 사용한다. (권장x)
* `@EntityListeners(AuditingEntityListener.class)`를 생략하고 글로벌로 적용하려면 orm.xml에 등록하면 된다.  

  
등록시간, 수정시간만 필요한 경우가 있을 수 도 있기 때문에 다음과 같이 Base 타입을 분리하고, 원하는 타입을 선택해서 상속한다.
```java
public class BaseTimeEntity {
   @CreatedDate
   @Column(updatable = false)
   private LocalDateTime createdDate;
   @LastModifiedDate
   private LocalDateTime lastModifiedDate;
}

public class BaseEntity extends BaseTimeEntity {
   @CreatedBy
   @Column(updatable = false)
   private String createdBy;
   @LastModifiedBy
   private String lastModifiedBy;
}
```

***

## Web 확장 - 도메인 클래스 컨버터
HTTP 파라미터로 넘어온 엔티티의 아이디로 엔티티 객체를 찾아서 바인딩 해주는 기능
### 도메인 클래스 컨버터 사용 전
```java
@GetMapping("members/{id}")
public String findMember(@PathVariable("id") Long id) {
    Member member = memberRepository.findById(id).get();
    return member.getUsername();
}
```
### 도메인 클래스 컨버터 사용 후
```java
@GetMapping("members/{id}")
public String findMember(@PathVariable("id") Member member) {
    return member.getUsername();
}
```
* HTTP 요청은 회원 id를 받지만 도메인 클래스 컨버터가 중간에 동작해서 회원 엔티티 객체를 반환
* 내부적으로 리포지토리를 사용해서 엔티티를 찾음
> **주의**  
도메인 클래스 컨버터로 받은 엔티티는 단순 조회용으로만 사용해야 한다. (detach 상태임)  

***

## Web 확장 - 페이징과 정렬
스프링 데이터가 제공하는 페이징과 정렬 기능을 스프링 MVC에서 편리하게 사용할 수 있게 해주는 기능  
```java
@GetMapping("members")
public Page<Member> list(Pageable pageable) {
    return memberRepository.findAll(pageable);
}
```  
* 위와 같이만 작성해도 실제 호출시 query 파라미터로 page, size, sort를 사용할 수 있다.  
* 예시) /members?page=0&size=3&sort=id,desc&sort=username,desc  

### 페이징 & 정렬 기본값 변경  
* 글로벌 설정: 스프링 부트
    ```yml
    spring.data.web.pageable.default-page-size=(기본페이지 사이즈 기본값: 20)
    spring.data.web.pageable.max-page-size=(최대 페이지 사이즈 기본값: 2000)
    ```
* 개별 설정 : `@PageableDefault`
    ```java
    @GetMapping("members")
    public String list(@PageableDefault(size = 12, sort = “username”, direction = Sort.Direction.DESC) Pageable pageable) {
    ...
    }
    ```
### 멀티 페이징
* 페이징 정보가 둘 이상이면 접두사로 구분
* `@Qualifier`에 접두사명 추가 "{접두사명}_xxx"
```java
public String list(
 @Qualifier("member") Pageable memberPageable,
 @Qualifier("order") Pageable orderPageable, ...
```
* 예시) /member?member_page=0&order_page=1 

***

## 스프링 데이터 JPA 구현체 분석
스프링 데이터 JPA 공통 인터페이스를 상속한 인터페이스가 동작하는 이유는  
애플리케이션 실행 시점에 스프링 데이터 JPA가 구현체를 직접 생성해주기 때문이다.  
그렇다면 실제 구현체는 무엇이고 어떻게 구현되어 있을까?  

### SimpleJpaRepository
```java
@Repository
@Transactional(readOnly = true)
public class SimpleJpaRepository<T, ID> ...{

     @Transactional
     public <S extends T> S save(S entity) {
         if (entityInformation.isNew(entity)) {
             em.persist(entity);
             return entity;
         } else {
             return em.merge(entity);
         }
     }
     ...
}
```
* 실제 구현체  
* `org.springframework.data.jpa.repository.support.SimpleJpaRepository`에 존재  
* `@Repository` 적용 : JPA 예외를 스프링이 추상화한 예외로 변환
* `@Transactional`이 이미 적용되어 있음
  * 서비스 계층에서 트랜잭션을 시작하지 않아도 리포지토리에서 트랜잭션이 시작
  * 서비스 계층에서 트랜잭션을 시작하면 리포지토리는 해당 트랜잭션을 전파 받아서 사용
  * `readOnly = true` 적용 : 내부적으로 성능 최적화를 하고 있음
* `save()` 메서드 : **새로운 엔티티면 저장 아니면 병합**

***

## 새로운 엔티티를 구별하는 방법
스프링 데이터 JPA 공통 인터페이스의 `save(S)` 메서드는 **새로운 엔티티면 저장 아니면 병합**을 수행한다.  
그렇다면 새로운 엔티티를 판단하는 기준은 무엇일까?  

### 새로운 엔티티를 판단하는 기본 전략
* 식별자가 객체일 때 `null`로 판단 
* 식별자가 자바 기본 타입일 때 `0`으로 판단
* `Persistable` 인터페이스를 구현해서 판단 로직 변경 가능

> **참고**  
JPA 식별자 생성 전략이 `@GenerateValue`면 save 호출 시점에 식별자가 없으므로 새로운 엔티티로 인식해서 정상 동작한다.  
하지만 식별자를 직접 할당하면 이미 식별자 값이 있는 상태로 save를 호출하기 때문에 merge가 호출된다.(비효율적)  
따라서 `Persistable`를 사용해서 새로운 엔티티 확인 여부를 직접 구현하는게 효과적이다.  

### Peresistable 구현
```java
@Entity
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@EntityListeners(AuditingEntityListener.class)
public class Item implements Persistable<String> {

     @Id
     private String id;
     
     @CreatedDate
     private LocalDateTime createdDate;
     
     public Item(String id) {
     this.id = id;
     }
     
     @Override
     public String getId() {
       return id;
     }
     
     @Override
     public boolean isNew() {
       return createdDate == null;
     }
}
```
* `getId()`와 `isNew()`를 상속받아서 구현한다.  
* Auditing을 이용해 생성 날짜로 새로운 엔티티 여부를 판단할 수 있다.  

***


