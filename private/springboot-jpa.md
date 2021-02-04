# 스프링부트와 JPA

## 스프링부트와 application.yml
* 스프링부트 테스트를 할때 `test/resources` 경로에 `application.yml`을 추가하면 해당 파일을 설정파일로 사용한다.
* 스프링부트는 `dateasource` 설정이 없으면, 기본적으로 `메모리 DB`를 사용한다.
* `driver-class`도 현재 등록된 라이브러리를 보고 찾아준다.
* `ddl-auto`도 `create-drop` 모드로 동작한다.  

따라서 데이터소스나, JPA 관련된 별도의 추가 설정없이 파일만 생성해 놓아도 자동으로 셋팅돼서 동작한다.  

## 스프링부트와 테이블, 컬럼명 생성 전략
하이버네이트는 엔티티의 필드명을 테이블 명으로 사용하지만  
스프링부트는 하이버네이트의 기본 매핑 전략을 변경해서 사용한다.  
* 카멜 케이스 -> 언더스코어 (memberPoint -> member_point)
* .(점) -> \_(언더스코어)
* 대문자 -> 소문자  

이러한 매핑 전략은 사용자가 직접 커스텀해서 사용할 수 있다.  
### 논리명 생성
명시적으로 컬럼, 테이블명을 직접 적지 않으면 `ImplicitNamingStrategy`사용  
`spring.jpa.hibernate.nameing.implicit-strategy`: 테이블이나, 컬럼명을 명시하지 않을 때 논리명 적용
### 물리명 적용
`spring.jpa.hibernate.nameing.physical-strategy`: 모든 논리명에 적용됨, 실제 테이블에 적용
### 스프링 부트 기본 설정
`spring.jpa.hibernate.nameing.implicit-strategy`: `org.springframework.boot.orm.jpa.hibernamte.SpringImplicitNamingStrategy`  
`spring.jpa.hibernate.nameing.physical-strategy`: `org.springframework.boot.orm.jpa.hibernamte.SpringPhysicalNamingStrategy`

## OSIV (Open Session In View)
영속성 컨텍스트는 트랜잭션과 라이프사이클을 같이하기 때문에 `@Transactional`을 걸어준 메소드가 실행될때  
영속성 컨텍스트가 생성되면서 데이터베이스 커넥션을 가져오고 메소드가 종료되면 영속성 컨텍스트가 닫히고 커넥션을 반환한다.   
보통 Controller -> Service -> Repository와 같은 아키텍처를 구성하고  
Service 계층에 트랜잭션을 거는 경우가 많기 때문에 **서비스 계층을 벗어나 컨트롤러 계층으로 가게 되면 영속성 컨텍스트 닫힌다.**  
하지만 다음과 같이 `spring.jpa.open-in-view:true(기본값)` OSIV가 켜져있는 상태라면  
**영속성 컨텍스트가 컨트롤러는 물론 View Template까지 API 응답이 끝날 때 까지 영속성 컨텍스트와 데이터베이스 커넥션을 유지한다.**  
### OSIV ON
트랜잭션 밖의 컨트롤러나 View Template에서도 영속성 컨텍스트가 살아있기 때문에 지연로딩이 가능하고  
코드를 유연하게 작성할 수 있다는 장점이 있지만 **데이터베이스 커넥션을 오래 가지고 있기 때문에 커넥션이 마를 수 도 있다.**  
### OSIV OFF
트랜잭션 범위 안에서 지연로딩을 강제로 호출해 두어야 한다.  
코드가 복잡해진다.
### 조언 : 커맨드와 쿼리 분리  
OSIV를 끈 상태로 복잡성을 관리하려면 `Command`와 `Query`를 분리해야 한다.  
보통 복잡한 화면을 출력하기 위한 쿼리는 화면에 맞추어 성능을 최적화 하는 것 이 중요하다. 하지만 그 복잡성에 비해 핵심 비즈니스에 큰 영향을 주는 것은 아니다.  
그래서 둘의 관심사를 명확하게 분리한다.  
* OderService
  * OrderService : 핵심 비즈니스 로직
  * OrderQueryService : 화면이나 API에 맞춘 서비스 (주로 읽기 전용 트랜잭션 사용)



