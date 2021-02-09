# 스프링 데이터 JPA 확장 기능

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

