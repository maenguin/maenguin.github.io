# 스프링 데이타 JPA

## 리포지토리 별로 반복해서 작성했던 CRUD 코드를 작성할 필요가 없다


## 공통인터페이스 분석


### JpaRepository
`spring-data-jpa` 라이브러리의  
`org.springframework.data.jpa.respository` 패키지에 있는 인터페이스이다.  
Jpa에 특화된 인터페이스라고 보면된다.  
`PagingAndSortingRepository`와 `QueryByExampleExecuor` 인터페이스를 상속받았다.  


### PagingAndSortingRepository
`spring-data-commons` 라이브러리의  
`org.springframework.data.respository` 패키지에 있는 인터페이스이다.  
Jpa와 Rdb에 특화되어 있는 위와는 다르게 공통적인 속성으로 정의되어 있다.  
`CrudeRepository` 인터페이스를 상속받았다.  

### CrudeRepository
패키지는 위와 같다.  
기본적인 CRUD 기능이 정의되어 있다.  
`Repository` 인터페이스를 상속받았다.  

### Repository
패키지는 위와 같다.  
아무런 기능이 정의되어 있지 않고 마커 인터페이스 역할을 한다.  
최상단의 인터페이스로서 스프링 컴포넌트 스캔을 용이하게 해준다.  
