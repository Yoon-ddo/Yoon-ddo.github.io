---
title: KDT:::Sparta MSA 아키텍처 3기(week02)
toc: true
toc_sticky: true
toc_label: "Contents of Page"
categories:
  - Study
tags:
  - MSA_architecture
---

<br><br><br>

# 스프링의 핵심 원리
## `강한결합`의 문제점 
* 코드 내에서 직접 **new**를 사용해 객체를 생성
* 하나의 변경이 다른 코드의 변경을 유발하는 구조는 유지보수가 어렵다.
* 모듈간 의존성이 높아 테스트가 힘들다.
## Spring IoC 컨테이너와 IoC로 해결
### Spring IoC - 애플리케이션을 구성하는 객체(Bean)들의 생성, 관리, 연결을 총괄
* 객체 생성 및 관리 : 어노테이션을 읽어 객체를 생성하고 소멸까지 생성주기를 관리
* 의존성 연결 : 객체간의 관계를 파악하여 알맞게 연결
### IoC (Inversion of Control) - 제어의 역전
* Spring 컨테이너가 동작하는 핵심원리
* 과거에는 개발자가 직접 객체를 생성하고 제어했지만, 이제는 그 '제어권'을 Spring 컨테이너에게 역전(Inversion)시킴.
## Spring Bean - 컨테이너가 관리하는 객체의 단위
* Spring IoC 컨테이너가 직접 생성하고, 의존관계를 설정해주며, 생명주기까지 관리하는 객체
* 클래스에 `@Component`, `@Service`, `@Repository`, `@Controller` 등의 어노테이션을 붙인다.
* Spring이 애플리케이션 시작 시 해당 클래스를 스캔하여 IoC 컨테이너에 Bean으로 생성(등록)한다.
* @Component는 가장 기본적인 어노테이션이며, 나머지는 특정 목적을 가진 특화된 형태이다.
## DI (Dependency Injection) - IoC를 구현하는 기술
* 클래스가 필요로 하는 의존 객체(Dependency)를 Spring 컨테이너가 자동으로 찾아서 연결(주입, Injection)해준다.
* 강한결합, DI 코드비교

```Java
// 강한 결합의 예시
public class UserController {
    // UserService가 UserRepository의 존재를 직접 알고, 직접 생성함
    private final UserService userService = new UserServiceImpl();
   // UserServiceImpl를 NewUserServiceImpl로 변경해야 한다면, UserController 클래스의 코드를 직접 수정해야 한다.

    public void save() {
        userService.save();
    }
}

// DI를 통해 느슨한 결합이 구현된 예시
public class UserController {
   
   //'어떤' Service인지는 모르지만, UserService타입의 Bean이 필요하다고 선언만 함
    private final UserService userService;


   // 생성자를 통해 Spring 컨테이너가 알아서 적절한 Bean을 '주입'해준다.
    public UserController(UserService userService) {
        this.userService = userService;
    }
}

// 스프링컨테이너 빈 등록 
@Configuration
public class BeanConfig {

  @Bean
  public UserService userService() {
    return new UserServiceImpl(); // UserServiceImpl를 NewUserServiceImpl로 바꾸고 싶어도, UserService의 코드는 단 한 줄도 수정할 필요가 없어진다.
  }
}
```

<br><br>

# 의존성 주입(DI)의 3가지 방식
## 생성자주입 - UserController를 만들 때, 필요한 부품(UserService)을 생성자에서 바로 넣어줌.
* 📘 개념
  - 가장 권장되는 방식
  - 의존 객체를 생성자 매개변수로 전달받는 방식
  - final 키워드와 함께 쓰면 불변성을 보장할 수 있다.
* ✅ 예시
```Java
@Component
public class UserController {

    private final UserService userService; // final로 선언

    // 생성자 주입
    public UserController(UserService userService) {
        this.userService = userService;
    }

    public void getUser() {
        userService.findUser();
    }
}
```
* 🌟 장점
  - 불변성 보장: final로 선언해 한 번 주입된 후 변경 불가
  - 필수 의존성 보장: 생성자 호출 시 반드시 주입되어야 하므로 NPE 위험 낮음
  - 테스트 용이: 직접 mock 객체를 생성자에 주입할 수 있음
  - 순환 참조 감지 가능: 스프링이 컴파일 시점에 감지 가능
* ⚠️ 단점
 - 의존성이 너무 많을 때(예: 5개 이상) 생성자가 길어질 수 있음  
## 수정자주입 - UserController를 먼저 만들고, 나중에 필요한 부품을 세터 메서드로 꽂아줌.
* 📘 개념
  - 의존 객체를 setter 메서드(수정자) 를 통해 주입받는 방식
  - 기본 생성자로 객체를 만들고, 나중에 setUserService() 같은 메서드로 주입
* ✅ 예시
```Java
@Component
public class UserController {

    private UserService userService; 

    /* setter 메서드를 통해 주입 
      👉 객체가 만들어진 다음에
      👉 setUserService()라는 메서드를 이용해서
      👉 외부(Spring 컨테이너)가 의존 객체를 집어넣어주는 것
    */
    @Autowired
    public void setUserService(UserService userService) {
        this.userService = userService; // 객체가 만들어진 뒤에 값이 들어오므로 private final UserService userService; 사용불가.
    }

    public void getUser() {
        userService.findUser();
    }
}
```
* 🌟 장점
  - 선택적 의존성을 주입할 때 유용 (필수 아님)
  - 순환 의존성 문제 해결에 유리한 경우 있음
* ⚠️ 단점
  - 객체가 완전히 생성된 이후에 의존성이 주입되므로, 주입이 누락되면 런타임 오류 발생 가능
  - 불변성 보장 불가 (필드를 final로 못 씀)
  - 코드가 장황해질 수 있음
## 필드주입
* 📘개념
  - 의존 객체를 필드에 직접 주입하는 방식
  - @Autowired를 필드에 바로 붙입니다.
  - 예전 코드나 간단한 테스트용 코드에서 자주 사용됩니다.
* ✅ 예시
```Java
@Component
public class UserController {

    @Autowired
    private UserService userService; // 필드에 직접 주입

    public void getUser() {
        userService.findUser();
    }
}
```
* 🌟 장점
  - 코드가 간결하고 빠르게 작성 가능
  - 테스트나 간단한 예제에서 편리
* ⚠️ 단점
  - 의존 관계가 외부에서 보이지 않음 (캡슐화 깨짐)
  - 불변성 보장 불가
  - 순환 의존성 감지 어려움
  - 단위 테스트 시 Mock 객체 주입 어려움 ➡️ 그래서 실무에서는 비추천

## 총정리

| 구분 | 생성자 주입 | 수정자 주입 | 필드 주입 |
|------|--------------|--------------|------------|
| 주입 시점 | 객체 생성 시 | 객체 생성 후 | 런타임 시 |
| 불변성 | ✅ 보장 | ❌ 불가 | ❌ 불가 |
| 필수 의존성 보장 | ✅ 가능 | ❌ 불가 | ❌ 불가 |
| 코드 가독성 | 약간 복잡 | 보통 | 간결 |
| 테스트 용이성 | ✅ 매우 높음 | 보통 | ❌ 낮음 |
| 순환참조 감지 | ✅ 컴파일 시 | ⚠️ 제한적 | ❌ 어려움 |
| 스프링 권장 여부 | ✅ 적극 권장 | 보조적 사용 | ❌ 비추천 |

<br><br>

# 계층형 아키텍처 (Layered Architecture)
* `Client ↔ Controller ↔ Service ↔ Repository ↔ DB`
  - **Controller**: 클라이언트의 HTTP 요청을 가장 먼저 받는 '**진입점**' 요청을 분석하여 적절한 서비스 계층의 메서드를 호출하고, 그 결과를 클라이언트에게 응답하는 역할
  - **Service**: 애플리케이션의 핵심 '**비즈니스 로직**'을 처리 데이터 가공, 트랜잭션 관리 등 실제 업무 규칙이 구현되는 곳
  - **Repository**: 데이터베이스와 직접 통신하는 '**데이터 접근 계층**' JPA의 `JpaRepository`를 상속받아 데이터의 저장, 조회, 수정, 삭제를 담당
## 기본적인 Spring Boot 폴더 구조 (권장 패키지 구조)

| 구분 | 파일명 | 역할 |
|------|---------|------|
| **Controller** | `ProductController.java` | HTTP 요청을 받아 서비스 호출, 결과 반환 |
| **Service** | `ProductService.java` | 비즈니스 로직 담당 (예: 검증, 계산, 트랜잭션 처리 등) |
| **Repository** | `ProductRepository.java` | 데이터베이스와 직접 통신 (JPA 인터페이스) |
| **Entity** | `Product.java` | DB 테이블과 1:1 매핑되는 클래스 (필드 = 컬럼) |
| **Main** | `DemoApplication.java` | `@SpringBootApplication` 붙은 시작 클래스 |
| **Config** *(선택)* | `WebConfig.java`, `BeanConfig.java` 등 | Bean 설정, CORS, 인터셉터 등 추가 설정 |


```
src
 └─ main
     ├─ java
     │   └─ com.example.demo
     │       ├─ DemoApplication.java          ← 메인 클래스 (프로그램 시작점)
     │       │
     │       ├─ controller                    ← 요청(Request)을 받는 계층
     │       │    └─ ProductController.java   ← @RestController 작성 위치
     │       │
     │       ├─ service                       ← 비즈니스 로직 계층
     │       │    └─ ProductService.java
     │       │
     │       ├─ repository                    ← DB 접근 계층 (JPA 등)
     │       │    └─ ProductRepository.java
     │       │
     │       └─ entity                        ← DB 테이블과 매핑되는 도메인 객체
     │            └─ Product.java
     │
     └─ resources
         ├─ application.yml                   ← 설정 파일
         └─ static / templates (선택)         ← 정적 파일 or 템플릿
```

