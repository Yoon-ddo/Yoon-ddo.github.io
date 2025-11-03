---
title: KDT:::Sparta MSA 아키텍처 3기(week01)
toc: true
toc_sticky: true
toc_label: "Contents of Page"
categories:
  - Study
tags:
  - MSA_architecture
---

<br><br><br>

# 스프링을 사용하는 이유
* 스프링 프레임워크는 자바(Java) 기반의 애플리케이션을 개발하는 데 필요한 거의 모든 것을 지원하는 오픈소스 프레임워크이다.
* 웹 개발뿐만 아니라, 데이터 처리, 보안, 배치(batch) 작업 등 기업 환경의 복잡한 시스템을 구축하는 데 필요한 포괄적인 프로그래밍 및 설정 모델을 제공한다.

## 스프링 3대 핵심 특징
### 1. IoC (Inversion of Control) / DI (Dependency Injection): 제어의 역전 / 의존성 주입
* 객체의 생성과 생명주기 관리를 스프링 컨테이너가 대신해 줍니다. 개발자는 필요한 객체를 선언만 하면, 스프링이 알아서 주입(Injection)해준다.
### 2. AOP (Aspect-Oriented Programming): 관점 지향 프로그래밍
* 애플리케이션 전반에 걸쳐 공통적으로 적용되는 기능(예: 로깅, 보안, 트랜잭션)을 비즈니스 로직과 분리하여 모듈화한다.
* 메인 비즈니스 코드에는 영향을 주지 않으면서, 필요한 부분에 공통 기능을 끼워넣을 수 있다.
### 3. PSA (Portable Service Abstraction): 일관된 서비스 추상화
* 데이터베이스 접근 방식(JPA, JDBC 등), 트랜잭션 처리 등 다양한 기술 구현체들을 스프링이 제공하는 일관된 방식으로 사용할 수 있다.
* 기술이 바뀌더라도 코드 수정없이 바꿀 수 있다.

## 현재 실무에서 사용 중인 ProC와 다른점은?
|구분	|POJO (Plain Old Java Object)	|ProC |
|------|---|---|
|언어	|Java|C + SQL (Oracle 전용)|
|목적	|프레임워크에 독립적인 객체 지향 코드 작성	|C 코드 안에 SQL을 직접 포함해서 DB 연동|
|철학	|순수 자바 코드로 비즈니스 로직만 작성	|SQL을 C 프로그램에 삽입해서 성능 높은 DB 프로그램 작성|
|의존성	|프레임워크에 의존하지 않음	|Oracle ProC Precompiler에 의존|
|사용 맥락	|스프링, JPA, Hibernate 등에서 핵심 철학	|오래된 금융권/레거시 시스템의 DB 프로그래밍|

<br><br>

## 디렉토리 구조 예시
```java
src/main
     ├─ java/com/tmp
     │           ├─ TempApplication.java
     │           ├─ common # 📂 여러 도메인에서 공통으로 사용하는 코드
     │           ├─ domain # 📌 도메인별 패키지 
     │           │   ├─ category/Category.java     # 도메인명/데이터베이스 테이블과 매핑되는 객체클래스 .java
     │           │   ├─ product/Product.java
     │           │   ├─ order/Order.java
     │           │   └─ refund/Refund.java
     │           ├─ dto #데이터 전송 객체 (Request/Response)
     │           │   ├─ category/CategoryDtos.java
     │           │   ├─ product/ProductDtos.java
     │           │   ├─ order/OrderDtos.java
     │           │   └─ refund/RefundDtos.java
     │           ├─ repository # 데이터베이스 접근 로직 (JPA 인터페이스)
     │           │   ├─ CategoryRepository.java
     │           │   ├─ ProductRepository.java
     │           │   ├─ OrderRepository.java
     │           │   └─ RefundRepository.java
     │           ├─ service # 관련 비즈니스 로직 구현
     │           │   ├─ CategoryService.java
     │           │   ├─ ProductService.java
     │           │   ├─ OrderService.java
     │           │   └─ RefundService.java
     │           └─ web
     │               ├─ CategoryController.java
     │               ├─ ProductController.java
     │               ├─ OrderController.java
     │               └─ RefundController.java   
     └── resources
          ├── db/migration/            # DB 마이그레이션 스크립트 (Flyway, Liquibase) : 파일명명규칭 !! V<버전>__<설명>.sql 
          ├── application.yml          # 애플리케이션 주요 설정 파일
          └── static/                  # CSS, JS, 이미지 등 정적 리소스

```

<br><br>

# ORM
* 객체-관계 불일치' 문제를 해결하기 위해 등장한 기술이다.
* 객체와 DB 테이블을 자동으로 매핑하여, 개발자가 SQL 없이 객체 중심으로 데이터를 다룰 수 있게 해줍니다.
## ORM 장점
- **생산성 향상**: SQL보다 객체 중심의 코드로 비즈니스 로직에 집중할 수 있습니다.
- **유지보수 용이**: 객체 모델만 수정하면 되므로 관리가 편합니다.
- **DB 독립성**: 특정 데이터베이스에 종속되지 않는 코드를 작성할 수 있습니다.
- **고급 기능**: Hibernate는 지연 로딩(Lazy Loading), 캐싱 등 성능 최적화를 위한 고급 기능을 제공합니다.

# Spring Data JPA
* '벤더 종속성(Vendor Lock-in)' 문제가 있었습니다. 다른 ORM 기술로 전환하려면 코드를 전부 수정해야 하는 문제 해결
* @Entity, @Id 같은 어노테이션과 persist(), find() 같은 메서드를 표준으로 정의합니다.
* 실제 동작하는 코드가 아니라, ORM 프레임워크들이 따라야 할 설계도이다.
* **JPA (설계도)**: 데이터베이스 연동 기술에 대한 표준 명세(인터페이스)
* **Hibernate (실제 일꾼)**: JPA라는 설계도를 보고 실제로 구현한 가장 유명한 구현체
* **Spring Data JPA (편의 도구)**: Hibernate 같은 JPA 구현체를 더 쉽고 편하게 사용하도록 한 번 더 감싸서, `Repository` 인터페이스만으로도 개발이 가능하게 만든 스프링 프레임워크의 모듈

# Hibernate
* ORM 개념을 구현한 가장 대표적인 프레임워크
* 객체 중심 코드를 작성하면, Hibernate가 적절한 SQL을 생성하고 실행하여 복잡한 데이터 변환 과정을 처리한다.

<br><br>

# 데이터 로딩
* 객체지향 프로그래밍(OOP)과 관계형 데이터베이스(RDB)가 데이터를 바라보는 방식이 근본적으로 다르다
* 객체지향에서는 (.) 하나로 객체에 접근할 수 있다.
* 그러나, 데이터베이스에서는 테이블로 분리되어있고 참조를 위해서는 JOIN과 같은 별도의 연산이 필요하다.
## 딜레마
* 개발자가 userRepository.findById(1L)로 User 객체 하나를 요청했을 때,
* "User와 연관된 Purchase 데이터도 지금 당장 함께 가져와야 할까, 아니면 나중에 진짜 필요하다고 할 때 가져올까?"
## 로딩전략
### 즉시 로딩(Eager Loading) : 필요할때만 부른다.
* 혹시 모르니 무조건 함께 가져오자!" 라는 전략. `JOIN`을 사용해 한 번에 모든 데이터를 가져온다.
### 지연 로딩(Lazy Loading) : 무조건 함께 부른다.
* "일단 급한 것만 주고, 필요하다고 하면 그때 가서 가져다주자!" 라는 전략. `User`만 먼저 가져오고, `Purchase`는 나중에 별도 쿼리로 가져온다.
