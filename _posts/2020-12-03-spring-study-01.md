---
title: "스프링 코어"
categories:
  - spring
tags:
  - 스프링
---

# 스프링의 핵심 기능

IoC (Inversion of Control)
:   제어의 역전, 리소스 관리 역할을 유저가 아닌 프레임워크에 위임했다는 뜻이다. DI를 통해 리소스를 사용할 수 있다.

DI (Dependency Injection)
:   의존 관계 주입, 객체를 직접 생성하는것이 아니라 외부에서 주입받는것을 의미한다.

스프링 IoC Container
:   애플리케이션 컴포넌트의 중앙 저장소, POJO 인스턴스(bean)을 구성하고 관리해준다.

## 빈등록과 DI
* 스프링 컨터이너에 빈을 등록할때 기본적으로 싱글톤으로 등록한다.
* 스프링에서는 여러가지 빈 생성 방식과 주입 방법을 제공하는데 그 중에 대표적으로 사용되는 두가지가 있다. (xml도 있지만 잘 사용하지 않는다.)

### ComponentScan & Autowired
* POJO 클래스에 `@Component`를 붙이면 실행시 컴포넌트 스캔에 의해 빈이 자동으로 등록된다. (@Controller, @Service, @Repository도 있다.)
* 등록된 빈을 `@Autowired`를 사용해 주입받을 수 있다. (생성자, 필드, setter 주입중 생성자 주입을 권장한다.)
```java
@Service
public class AccountService {

   private final AccountRepository accountRepository;

   @Autowired
   public AccountService(AccountRepository accountRepository) {
       this.accountRepository = accountRepository;
   }
}
```

### Configration & Bean
* `@Configration`을 통해 스프링에게 빈설정 파일임을 알려줄 수 있다.
* `@Bean`을 붙인 메소드를 다음과 같이 작성하여 빈을 등록하면서 의존관계도 설정할 수 있다.

```java
@Configuration
public class SpringConfig {
   @Bean
   public AccountService accountService() {
      return new AccountService(accountRepository());
   }
   @Bean
   public AccountRepository accountRepository() {
      return new accountRepository();
   }
}
```


# 스프링 웹 개발
스프링을 사용해 웹 개발을 할때 기능을 크게 3가지로 나눌 수 있다.
* 정적 컨텐츠
* MVC와 템플릿 엔진
* API

## 정적 컨텐츠
* resources:static 폴더에 파일을 넣고 웹서버로서의 역할을 할 수 있다.
* resources:static/index.html이 존재하면 Welcompage로서 동작한다.

## MVC와 템플릿 엔진
* feemarker, thymeleaf등의 템플릿 엔진을 이용해 기존의 JSP와 같이 동적 컨텐츠를 생성하여 클라이언트에게 반환할 수 있다.
* 컨트롤러에서 문자열값을 반환하면 `ViewResolver`가 화면을 찾아서 처리한다.
```java
@GetMapping("hello")
public String hello(Model model){
    model.addAttribute("data","hello!!");
    return "hello"; //resources:templates/hello.html에 맵핑된다.
}
```


## API
* 컨트롤러에 `@ResponseBody`를 사용하는 경우 `HttpMessageConverter`가 동작하여 Http Body에 데이터가 그대로 주입되어 클라이언트에게 반환할 수 있다.
* 문자열을 반환하면 StringConverter가 동작해 문자열을 반환한다.
* 객체를 반환하면 기본적으로 JsonConverter가 동작해 Json 포맷형식으로 반환된다.
```java
@ResponseBody
@GetMapping("hello")
public String hello(){
    Hello hello = new Hello();
    hello.setName("Hi");
    return hello; // { "name" : "Hi"}
}
```




