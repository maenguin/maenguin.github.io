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




