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

<br><br>

# 기본적인 Spring Boot 폴더 구조 (권장 패키지 구조)

| 구분 | 파일명 | 역할 |
|------|---------|------|
| **Controller** | `ProductController.java` | HTTP 요청을 받아 서비스 호출, 결과 반환 |
| **Service** | `ProductService.java` | 비즈니스 로직 담당 (예: 검증, 계산, 트랜잭션 처리 등) |
| **Repository** | `ProductRepository.java` | 데이터베이스와 직접 통신 (JPA 인터페이스) |
| **Entity** | `Product.java` | DB 테이블과 1:1 매핑되는 클래스 (필드 = 컬럼) |
| **Main** | `DemoApplication.java` | `@SpringBootApplication` 붙은 시작 클래스 |
| **Config** *(선택)* | `WebConfig.java`, `BeanConfig.java` 등 | Bean 설정, CORS, 인터셉터 등 추가 설정 |


```bash
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

<br><br>

# RESTful API 설계
## RESTful 등장 전, RPC(Remote Procedure Call) 스타일이나 SOAP(Simple Object Access Protocol) 같은 방식으로 통신
* **RPC 스타일**: `/getUser`, `/deleteProduct`처럼 URI에 행위(동사)가 포함된 방식
  - 서버의 특정 함수를 그대로 호출하는 것과 같아, 클라이언트와 서버가 서로의 내부 구현에 강하게 결합(Tight Coupling)되는 문제가 있었습니다.
* **SOAP 프로토콜**: XML을 기반으로 한 매우 복잡하고 무거운 프로토콜
  -  규칙이 엄격하고, 메시지 구조가 방대하여 배우기 어렵고 오버헤드가 큼
## 🌐 RESTful API 핵심 원칙
> RESTful API는 단순히 HTTP를 사용하는 것이 아니라,  
> **리소스를 일관된 방식으로 표현하고 조작하는 아키텍처 스타일**입니다.

---

### 🧩 1️⃣ Uniform Interface (일관된 인터페이스)
모든 자원(Resource)에 대해 **일정한 규칙으로 접근**해야 합니다.
- 자원은 **URI로 표현**합니다.
- GET /users → 사용자 목록 조회
- GET /users/1 → 특정 사용자 조회
- POST /users → 사용자 생성
- PUT /users/1 → 사용자 전체 수정
- DELETE /users/1 → 사용자 삭제
- HTTP 메서드(GET, POST, PUT, DELETE, PATCH 등)를 자원 조작 의미로 사용  
- **명사 중심의 URI**, 일관성 있는 구조 유지  
> ✅ **효과:** 클라이언트가 API 구조를 쉽게 예측하고 일관된 방식으로 사용할 수 있음

---

### 🚫 2️⃣ Stateless (무상태성)
서버는 클라이언트의 상태(Session, Context)를 **유지하지 않습니다.**
- 요청마다 필요한 모든 정보를 포함해야 합니다. (예: 인증 토큰, 파라미터 등)  
- 서버는 이전 요청을 기억하지 않고, 각 요청은 독립적으로 처리됩니다.
> ✅ **효과:** 서버 확장성(Scalability) 향상, 부하 분산 용이

---

### 👩‍💻 3️⃣ Client–Server 구조 분리
클라이언트(프론트엔드)와 서버(백엔드)의 역할을 **명확히 분리**합니다.
- 클라이언트 → UI/UX 담당
- 서버 → 데이터와 비즈니스 로직 담당  
- 서로 독립적으로 개발·배포 가능  
> ✅ **효과:** 유지보수성 향상, 다양한 플랫폼(웹/모바일)에서 동일한 API 사용 가능

---

### ⚡ 4️⃣ Cacheable (캐시 가능성)

응답 데이터에 **캐시 정책을 명시**해 재사용할 수 있어야 합니다.
- HTTP 헤더(`Cache-Control`, `ETag`, `Last-Modified`)를 활용  
- 변경이 적은 리소스(예: 이미지, 상품 목록)는 캐싱 가능  
> ✅ **효과:** 네트워크 트래픽 감소, 응답 속도 향상

---

### 🧱 5️⃣ Layered System (계층화 구조)
클라이언트는 **서버 내부 구조(계층)** 를 몰라도 됩니다.
- 요청은 여러 중간 서버(프록시, 로드 밸런서, 인증 서버 등)를 거칠 수 있음  
- 각 계층은 독립적으로 구성 가능  
> ✅ **효과:** 보안 강화, 확장성 및 관리 용이성 향상

---

### 💾 6️⃣ Code on Demand *(선택적)*
서버가 클라이언트에게 **실행 가능한 코드(JavaScript 등)** 를 내려보낼 수 있습니다.  
하지만 보안 및 제약 문제로 **실제 REST API에서는 거의 사용되지 않습니다.**

---

### ⚙️ 요약 표

| 원칙 | 의미 | 핵심 포인트 |
|------|------|-------------|
| **Uniform Interface** | 일관된 자원 표현 | URI는 명사, HTTP 메서드로 동작 구분 |
| **Stateless** | 무상태성 | 요청마다 필요한 모든 정보 포함 |
| **Client–Server** | 역할 분리 | UI와 비즈니스 로직 분리 |
| **Cacheable** | 캐시 가능 | 응답에 캐시 정책 명시 |
| **Layered System** | 계층 구조 | 프록시, 게이트웨이 등 중간 계층 허용 |
| **Code on Demand** *(선택)* | 실행 코드 전송 | 실제로는 거의 사용되지 않음 |

---

> 💡 **정리:**  
> RESTful API는  
> - “리소스(자원)” 중심으로  
> - “HTTP 메서드”를 활용해  
> - “일관되고 상태 없는 방식”으로 설계된 API입니다.

---

<br><br>

#JPA 영속성 컨텍스트
* 엔티티(Entity)를 영구 저장하는 환경이라는 뜻
* 실제로는 애플리케이션과 데이터베이스 사이에서 엔티티를 담아두고 관리하는 논리적인 공간(메모리 캐시)
* EntityManager를 통해 이 공간에 접근하고 관리
## 엔티티의 4가지 상태 (Lifecycle)
* 비영속
* 영속
* 준영속
* 삭제
## 장점
* 1차캐시
* 동일성보장
* 쓰기지연
* 변경감지

<br><br>

# 트랜잭션
* 트랜잭션은 이 두 작업을 '하나의 논리적인 작업 단위'로 묶어, "두 작업이 모두 성공하거나, 하나라도 실패하면 모두 없던 일로 만든다(롤백)"는 것을 보장
* 이를 통해 어떤 상황에서도 데이터의 일관성과 무결성이 지켜진다.
## @Transactional
- **동작 원리: AOP와 프록시(Proxy)**
    - `@Transactional`이 붙은 메서드를 호출하면, Spring은 **프록시(Proxy)**라는 가상의 대리 객체를 통해 해당 메서드를 실행.
    - **프록시의 역할**:
        1. 메서드 실행 전, 데이터베이스에 **트랜잭션을 시작** (`BEGIN TRANSACTION`)
        2. 실제 메서드 로직을 실행
        3. 메서드가 예외 없이 성공적으로 종료되면, 트랜잭션을 **커밋(Commit)**하여 모든 변경사항을 DB에 영구 저장
        4. 메서드 실행 중 `RuntimeException` 등 예외가 발생하면, 트랜잭션을 롤백(Rollback)하여 모든 변경사항을 취소하고 원래 상태로 되돌린다.

```Java
@Service
@RequiredArgsConstructor
public class PurchaseService {
    private final PurchaseRepository purchaseRepository;
    private final ProductRepository productRepository;

    @Transactional
    public void placeOrder(OrderRequest orderRequest) {
        // 1. 주문 생성 로직 (DB 변경 1)
        Purchase purchase= new Purchase(purchaseRequest);
        purchaseRepository.save(order);
    
        // 2. 재고 감소 로직 (DB 변경 2)
        Product product = productRepository.findById(orderRequest.getProductId())
            .orElseThrow(() -> new EntityNotFoundException("상품 없음"));
        product.decreaseStock(orderRequest.getQuantity());

        // 3. 만약 재고가 음수가 되면 예외 발생
        if (product.getStock() < 0) {
            throw new IllegalArgumentException("재고가 부족합니다.");
        }
    }
}
```
* Checked Exception에 대해서는 롤백하지 않으므로, 필요시 rollbackFor 옵션을 사용해야 한다.

<br><br>

# Dirty Checking과 Flush
## Dirty Checking 이란?
* 영속성 컨텍스트가 관리하는 엔티티의 변경 사항을 감지하여, UPDATE SQL을 자동으로 생성하고 실행해주는 JPA의 매우 강력하고 핵심적인 기능
* **동작 원리: 스냅샷과의 비교**
* JPA는 "스냅샷" 비교를 통해 이 '마법' 같은 기능을 구현합니다.
  - 1. **조회 및 스냅샷 생성**: 트랜잭션 안에서 엔티티를 처음 조회하면(`findById()`), JPA는 해당 엔티티를 1차 캐시에 저장할 때 **그 상태 그대로의 복사본(스냅샷)**을 함께 저장.
  - 2. **엔티티 상태 변경**: 개발자가 코드에서 `setter` 등을 호출하여 영속 상태의 엔티티 필드 값을 변경. 이 변경은 우선 메모리에 있는 객체에만 반영된다.
  - 3. **변경점 비교**: 트랜잭션 커밋 등으로 **`Flush`가 호출되는 시점**에, JPA는 1차 캐시에 있는 모든 엔티티의 현재 상태와 최초에 저장해둔 스냅샷을 필드별로 일일이 비교한다.
  - 4. **UPDATE 쿼리 자동 생성**: 만약 두 상태가 다르다면(즉, 변경이 감지되면), JPA는 변경된 내용을 바탕으로 `UPDATE` 쿼리를 자동으로 생성하여 '쓰기 지연 SQL 저장소'에 등록한 뒤 데이터베이스에 전송
## Flush란?
* 영속성 컨텍스트의 변경 내용을 데이터베이스에 동기화하는 작업
* '쓰기 지연 SQL 저장소'에 쌓여있던 SQL 쿼리들을 실제 DB로 전송한다.

