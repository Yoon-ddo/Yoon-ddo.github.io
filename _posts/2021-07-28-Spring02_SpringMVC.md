---
title: Spring02_MVC
toc: true
toc_sticky: true
toc_label: "Contents of Page"
categories:
  - Study
tags:
  - Spring_Study
---

<br><br>

# Spring MVC
* Spring MVC를 하기위해서는 `DI`문법과 AOP기술을 알아야함
  - DI : 객체 주입 기술
  - AOP : 코드 주입 기술

<br><br>

## 1. 용어알기
### 1-1. IoC
* Inversion of Control : 제어역행( 먼저 객체 생성 후, 프로그램 실행하면서 객체 사용 )
  - 기존에는 필요한 객체가 만들어지는 순간을 개발자가 제어함.(자기가 지우고, 만들고...)
  - IoC는 생명주기 관리를 개발자가 아닌 컨테이너가 처리.

* 스프링 컨테이너 : 만들어진 객체 보관소.

### 1-2. DI
* Dependency Injection : 의존주입 ( 외부에서 만들어진 객체를 가져와서 주입 )
* 내가 직접 `new`해서 사용X
* Spring에서 IoC를 제공하는 형태 중 하나 (DL, DI)
* 종류
  - Setter Injection (set메소드로 받아오는 방법)
  - Constructor Injection (생성자를 통해 받아오는 방법)
* 의존 : 객체간 의존관계 의미 

### 1-3. Container
* Spring에서 Container기능을 제공해주는 클래스를 의미
* Container : Bean 클래스를 관리 (생성, 삭제)하는 주체
* Bean : Spring에서 관리되는 클래스 객체를 나타냄 `<bean>`
* Container초기화 : 설정정보 xml파일을 읽고 Container에 로딩
* 종류
  - BeanFactory
  - ApplicationContext
* Bean
  - Spring Framework에 의해 생명주기가 관리되는 클래스
  - 일반 POJO기반의 클래스
  - XML에 <bean>태그 이용하여 등록
  - 속성 (보통 id, class사용)
    + id : bean클래스 식별하기 위한 이름설정(숫자가 우선할 수 없고 `/`와 같은 특수기호 사용 불가)
    + name
    + class : 사용하려는 bean클래서의 패키지명을 포함한 클래스명

<br><br><br><br>
  



