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
