# 스프링부트와 JPA

## 스프링부트와 application.yml
* 스프링부트 테스트를 할때 `test/resources` 경로에 `application.yml`을 추가하면 해당 파일을 설정파일로 사용한다.
* 스프링부트는 dateasource 설정이 없으면, 기본적으로 메모리 DB를 사용한다.
* driver-class도 현재 등록된 라이브러리를 보고 찾아준다.
* ddl-auto도 create-drop모드로 동작한다.  
따라서 데이터소스나, JPA 관련된 별도의 추가 설정없이 파일만 생성해 놓아도 자동으로 셋팅돼서 동작한다.  
