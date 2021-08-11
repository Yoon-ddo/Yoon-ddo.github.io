---
title: Spring06_복습(DI와 AOP)
toc: true
toc_sticky: true
toc_label: "Contents of Page"
categories:
  - Study
tags:
  - Spring_Study
---

<br><br>

* 이 게시물은 `처음 시작하는 스프링 프레임워크 - 허진경 지음` 책을 공부하며 작성한 글입니다.
  - <span style="color:lightgray">교수님! 좋은 책 주셔서 감사합니다:)</span>

# 1. DI 
## 1-1. DI
* DI를 사용하지 않는 코드는 직접 필요한 객체를 생성해서 사용해야했다.
 - HelloController.java : `IHelloService helloService = new HelloService();`
 - HelloController는 인터페이스를 이용해 기능을 사용하고 싶어도 HelloService클래스가 없다면 사용 불가. (강한 의존관계)
 - 강한 의존관계를 이루지 않도록 HelloService 클래스가 완성된 후 객체를 생성하여 HelloController클래스에게 줄 수 있음

* DI를 사용한 코드
  - `생성자 주입` : HelloController.java
    1. `IHelloService helloService;`를 멤버객체로 선언 후
    2. 생성자 주입 : `public HelloController(IHelloService helloservice){ this.helloService = helloservice; }`
    3. main에서 객체 생성시 생성자 주입
  - `Setter주입` : HelloController.java
    1. `IHelloService helloService;`를 멤버객체로 선언 후
    2. Setter주입 : `public void setHelloService(IHelloService helloservice){ this.helloService = helloservice; }`
    3. main에서 기본 생성자로 Controller객체 선언 후, .setHelloService(helloservice); 객체 전달

<br><br><br>

## 1-2. Spring DI
* 스프링의 의존성 주입?
  - 스프링 컨테이너
  - 스프링은 필요로하는 bean을 생성하여 컨테이너에서 관리한다.
  - 이때 bean을 생성하기 위한 설정과 의존객체를 주입시키기 위한 설정이 필요하다.
* 종류
  - XML을 이용한 DI
  - Annotation을 이용한 DI 

<br><br>

### 1-2-1. XML 이용한 DI
* 생성자를 이용한 의존성 주입
* setter 메소드를 이용한 의존성 주입

#### 00. 시작 전, 설정하기
* `src/main/resources`에서 Spring Bean Configuration File 생성 ( = xml 파일생성)
* bean설정
  - name / id : 고유 식별자 지정.
  - class : 필수속성, Bean작성하는데 사용할 클래스. `패키지이름.클래스이름`
  - scope : 객체 범위 (prototype(요청할때마다 새로운객체 생성), singleton(bean이 항상 하나만 생성), request, session (WebApplicationContext사용시 적용))
  - lazy-init : bean이 사용되는 시점에 인스턴스 생성 (true / false)
  - destroy-method : bean삭제 직전 지정한 메소드 실행

* `<import>`태그
  - import 태그를 이용하여 여러 설정 파일을 포함시킬 수 있음 ( 여러 파일을 한곳에 쓴것과 같은 기능 )

```xml
<import resource="context-common1.xml" />
<import resource="context-common2.xml" />
```

#### 01. Spring Context
  - 객체를 생성하고 관리하는 컨테이너 기능
  - BeanFactory 인터페이스를 구현한 클래스
    + Lazy Loading방식 : 클라이언트 요청시 bean생성
    + 설정파일에 등록된 bean을 생성하고 관리하는 기본적인 컨테이너 기능만 제공
  - ApplicationContext 인터페이스를 구현한 클래스
    + Pre Loading방식
    + bean을 생성하고 관리하는 기능 외에 트랜잭션 관리, 국제화 처리 등 많은 기능 제공. 
    + FileSystemXmlApplicationContext : 파일 시스템 경로의 xml설정 파일을 로딩
    + ClassPathXmlApplicationContext : 클래스 패스 경로에 있는 xml설정 파일 로딩
    + GenericXmlApplicationContext : 파일 시스템 경로의 xml, 클래스 패스 경로의 xml 파일 로딩 (클래스패스 경로 시 `classpath:` 접두어 붙인다.)
    + XmlWebApplicationContext : 웹 애플리케이션 개발에 사용되지만 직접 생성하지 않는 Context

```java
ApplicationContext context = new GenericXmlApplicationContext("application-config.xml");
context.close();
```

<br><br>

#### 02. 생성자 주입
* Controller 클래스에 IHelloService helloService; 인터페이스 객체 멤버변수 선언후 이에 객체를 주입하는 생성자 추가.
* 스프링 설정 파일에 `<constructor-arg>`태그로 의존성 주입
  - ref속성으로 의존성 주입할 다른 bean 아이디 지정 
  - <constructor-arg ref="helloService"/>

```xml
<bean id="helloService" class="패키지명.HelloService(클래스명)"/>
<bean id="helloController" class="패키지명.HelloController">
   <constructor-arg ref="helloService"/>
</bean>
```

<br>

* 생성자의 인자로 전달하는 인자의 type이 문자열이거나 기본 데이터 타입일 경우 value속성 이용

```xml
<constructor-arg value="oracle.jdbc.driver.OracleDriver" />
<constructor-arg value="jdbc:oracle:thin:@localhost:1521:xe" />
```

<br>

* index속성 지정도 가능.

```xml
<constructor-arg index="1" value="oracle.jdbc.driver.OracleDriver" />
<constructor-arg index="0" value="jdbc:oracle:thin:@localhost:1521:xe" />
```

<br>

* 인자로 전달하는 객체가 null일 경우 

```xml
<constructor-arg>
  <null/>
</constructor-arg>
```

<br><br>

#### 03. Setter메소드 주입
* Controller 클래스에 IHelloService helloService; 인터페이스 객체 멤버변수 선언후 이에 객체를 주입하는 Setter 메소드 추가.
* 스프링 설정파일에서 `<property>`태그를 이용하여 의존 객체 주입 <span style="color:red">( 기본 생성자 외에 다른 생성자 정의 금지 )</span>
  - name : 의존성을 주입할 bean의 고유 이름
  - ref : 의존성을 주입할 객체의 이름
  - value : 값을 지정

```xml
<bean id="helloService" class="패키지명.HelloService"/>
<bean id="helloController" class="패키지명.HelloController">
    <property name="helloService" ref="helloService" />
</bean>
```

<br>

* value로 string,기본데이터 타입 의존성 주입 가능

```xml
<bean id="helloController" class="패키지명.HelloController">
  <property name="driverClassName" value="oracle.jdbc.driver.OracleDriver" />
  <property name="url" value="jdbc:oracle:thin:@localhost:1521:xe" />
</bean>
```

<br>

* p네임스페이스
  - Setter주입 사용시 p네임스페이스를 이용하여 bean 태그의 속성으로 의존성 주입 가능.
  - Namespace탭에서 `p 네임스페이스` 선택. 별도의 스키마가 없으므로, 네임스페이스만 선언하고 사용할 수 있다.
  - 의존성 주입
    + `p:변수명-ref="bean이름"`
  - string과 기본 데이터 타입
    + `p:변수명="값"`
* NameSpace탭 생기게 하기
  - 난 일단 안쓰는 중.
  - 일반 xml 파일로 만들면 안뜨는 것같다.
  - Spring Config File 의 xml만 사용가능
  - [설치참고블로그](https://m.blog.naver.com/CommentList.naver?blogId=sulin00&logNo=221963921025)


```xml
<bean id="helloService" class="패키지명.HelloService" />
<bean id="helloController" p:helloService-ref="helloService" class="패키지명.HelloController" />
```

<br><br>

#### 04. Collection타입 의존성
* List, Set, Map, props
* 의존성 설정 태그

| 태그 | 타입 |
|:---:|:---:|
| \<list> | java.util.List 또는 배열 |
| \<set> | java.util.Set |
| \<map> | java.util.Map |
| \<props> | java.util.Properties |

<br>
  
* list

```xml
<property name="lists">
  <list>
    <value>1</value>
    <ref bean="personBean" />
    <bean class="패키지명.Person">
      <property name="name" value="HyunJeong"/>
      <property name="age" value="12"/>
    </bean>
  </list>
</property>
```

<br>
  
* set

```xml
<property name="sets">
  <set>
    <value>2</value>
    <ref bean="personBean" />
    <bean class="패키지명.Person">
      <property name="name" value="HyunJeong"/>
      <property name="age" value="12"/>
    </bean>
  </set>
</property>
```

<br>
  
* map

```xml
<property name="maps">
  <map>
    <entry key="key1" value="3"/>
    <entry key="key2" value-ref="personBean"/>
    <entry key="key3">
      <bean class="패키지명.Person">
        <property name="name" value="HyunSoo"/>
        <property name="age" value="8"/>
      </bean>
    </entry>
  </map>
</property>
```

<br>
  
* props

```xml
<property name="props">
  <props>
    <prop key="webmaster">webmaster@naver.com</prop>
    <prop key="support">support@naver.com</prop>
  </props>
</property>
```

* 사용하기

```java
// 1. xml 파일 불러오기
ApplicationContext context = new GenericXmlApplicationContext("application-config.xml");
// 2. bean태그로 선언한 객체 
Customer cust = context.getBean(Customer.class);
```

<br><br>

#### 05. Prototype과 Singleton
* 싱글톤 패턴이란?
  - 생성자가 여러 차례 호출되더라도 실제로 생성되는 객체는 하나이고 최초 생성 이후에 호출된 생성자는 최초의 생성자가 생성한 객체를 리턴한다. 
  - 주로 공통된 객체를 여러개 생성해서 사용하는 DBCP(DataBase Connection Pool)와 같은 상황에서 많이 사용된다.


* 스프링 컨테이너는 Bean 생성 시, 컨테이너에 클래스 당 한 개의 인스턴스만 생성한다.
* 클래스에 Singleton패턴을 적용하지 않아도 항상 한 개의 인스턴스만 생성
* Spring의 Singleton패턴은 자바의 클래스에 Singleton패턴을 적용하여 구현하지 않는다.
* Spring에서는 `싱클톤 레지스트리`를 이용하여 일반 클래스도 싱글톤처럼 관리해주는 방식 제공.
* bean태그의 scope 속성으로 bean이 싱글톤으로 생성되게 할지 아니면 요청할때마다 생성되게 할지 설정할 수 있다.
* scope 속성 값

| scope 속성 값 | 설명 |
|:---:|:---|
| singleton | 컨테이너에 한 개의 인스턴스만 생성. (기본값) |
| prototype | 빈을 요청할 때마다 인스턴스 생성. |
| thread | 스레드별 생성. 현재 실행중인 스레드에 종속. 스레드가 죽으면 bean도 소멸 |
| request | HTTP 요쳥마다 빈 객체 생성. WebApplicationContext에서만 적용 |
| session | HTTP 세션마다 빈 객체 생성. WebApplicationContext에서만 적용 |
| application | Singleton 과 유사. java.servlet.ServletContext에도 등록됨. |
| globalSession | 글로벌 HTTP세션에 대한 bean 객체 생성. Portlet을 지원하는 컨텍스트에만 적용 가능<br>글로벌 세션이 없으면 Session 과 기능이 같다. |


* 예제

```java
ApplicationContext context = new GenericXmlApllication("application-config.xml");
Person person1 = context.getBean(Person.class);
Person person2 = context.getBean(Person.class);
System.out.println(person1 == person2);
```

* scope에 `prototype`로 설정하고, 두개의 클래스를 이름을 다르게 해서 선언한 후 == 비교시 `false`가 출력된다.
  - prototype으로 설정할 때에는 bean을 요청한 클라이언트마다 자신의 상태 값을 가져야 할 때이다.
* scope에 `singleton`로 설정하고, 두개의 클래스를 이름을 다르게 해서 선언한 후 == 비교시 `true`가 출력된다. 

<br><br><br>

### 1-2-2. Annotation을 이용한 생성자 주입
#### 00. 설정하기
* `<context:component-scan>`태그로 설정가능
* bean으로 등록될 클래스들이 있는 패키지를 지정.
* 상위 패키지 지정 시 하위 패키지까지 bean으로 등록될 클래스로 
* Namespace이용하면 자동추가됨.

```xml
<context:componetn-scan base-package="패키지명"/>
```

* 설정 후, 패키지 내의 각각의 클래스에 어노테이션을 설정

<br><br>

#### 01. Bean 설정
* Class명 위에 붙인다.
1. @Component
  - 일반적인 컴포넌트로 등록되기 위한 클래스에 사용.
2. @Controller
  - 컨트롤러 클래스에 사용
3. @Service
  - 서비스 클래스에 사용
4. @Repository
  - DAO클래스 또는 Repository클래스에 사용

<br><br>

#### 02. @Autowired를 이용한 의존성 주입
* `@Autowired`는 Annotation Type을 기준으로 의존성 주입.
* 맴버객체 위에 붙이기
* 변수이름과 같은 클래스가 주입된다.

```java
@Autowired
IHelloService helloService; // HelloService주입
```

<br><br>

#### 03. @Qualifier를 이용한 의존객체 설정.
* 전달하고 싶은 객체를 설정할 수 있다.
* IHelloService라는 인터페이스에 HelloService가 아닌 NiceService라는 객체를 주입하고 싶은 경우
* NiceService @Service에 ("이름지정") 안했을 경우 자동으로 `niceService`라는 이름이 부여됨.
  - `niceService` : 앞글자 소문자로 바뀐 클래스 이름

```java
@Autowired
@Qualifier("niceService")
IHelloService helloService;
```

<br><br>

#### 04. @Resource를 이용한 의존객체 설정.
* @Autowired와 @Qualifier를 같이 사용하는 것과 같다.
* name 속성을 이용하여 bean의 이름을 직접 지정할 수 있다.
* Java 9 버전 사용한다면 사용불가.

```java
@Resource(name="niceService")
IHelloService helloService;
```

<br><br>

#### 05. @Inject를 이용한 의존성 주입
* @Autowired를 사용하는 것과 같다.
* 같은 타입의 bean이 두개 이상 있을 경우 변수의 이름과 같은 이름을 갖는 bean을 찾는다.
* 존재하지 않을 경우 에러발생 : BeanCreationException, NoSuchBeanDefinitionException)
* 

```java
@Inject
IHelloService helloService;
```

<br><br><br>

### 1-2-3. Xml과 Annotation 비교
#### 01. Bean생성
* XML
  - `<bean id="bean이름" class="패키지.클래스명"/>

* Annotation
  - 설정파일에 `<context:component-scan base-package="패키지명"/>` 추가
  - 자바 클래스 위에 @Controller, @Component, @Service, @Repository 중 하나 선언
  - bean의 이름은 클래스 이름에서 첫 문자만 소문자로 바뀐 이름으로 자동 지정

<br>

#### 02. 의존성 주입
* XML
  - 생성자
    + 자바 클래스에 생성자 추가
    + `<constructor-arg name="변수명" ref="bean이름" />`
  - Setter메소드
    + 자바 클래스에 메소드 추가
    + `<property name="변수명" ref="bean이름" />`

* Annotation
  - 생성자
    + 자바 클래스 필드, 생성자, Setter메소드 위에 `@Autowired` 또는 `@Inject` 중 하나 선언
  - Setter
    + 인터페이스를 구현한 클래스가 두 개 이상이면 `@Autowired`아래에 `@Qualifier("빈이름")`추가로 선언
    + `@Resource(name="빈이름")`으로 선언도 가능하지만 Java 9버전 사용불가 

---

<br><br><br><br>

# 2. AOP
* 코드주입
* 공통코드와 핵심코드의 분리
* 핵심로직을 구현한 코드에서 공통 기능을 직접적으로 호출하지 않음.

## 2-1. Proxy 클래스를 이용한 AOP 구현
* Proxy클래스 만들기

```java
@Component
public class HelloServiceProxy extends HelloService{
  
  @Override
  public String sayHello(String name) {
    //...
  }
  
  @Override
  public String sayGoodbye(String name){
   //...
  }
}
```

<br>

* Controller에서 의존객체 주입

```java
@Controller
public class HelloController{

  @Autowired
  @Qualifier("helloServiceProxy")
  IHelloService helloService;
  
  //...

}
```

* 매번 proxy를 만드는 것은 개발자에게 부담이 되는 일
* 부가적 코드를 만들지 않도록 하는 것이 AOP의 사용 목적.

### 2-1-1. AOP용어


| 용어 | 설명 | 예시 |
|:---:|:---|:---:|
| Joinpoint | 애플리케이션 실행 시, 특정 작업이 시작될 수 있는 시점을 의미<br> 클래스 로드시점, 인스턴스 생성시점, 메소드 호출 시점, 예외가 발생하는 시점 등이 해당됨<br>Advice를 적용할 수 있는 `시점`<br>스프링에서는 Proxy기반 AOP를 지원하기 때문에 메소드 호출 Joinpoint만 지원 | 모든 biz() 메소드 (sayHello(), sayGoodbye()) |
| Aspect | 여러 객체에 공통으로 적용되는 공통 관심 객체 | HelloLog 객체 |
| Advice | 핵심 코드에 삽입되어 동작할 수 있는 공통코드와 시점<br>before, after, after-throwing, after-returning, around 가 있다. | ~ 전(before)에 log() 메소드 실행 |
| Pointcut | JointPoint의 부분집합<br>실제 Advice가 적용되는 Joinpoint를 의미<br>Spring에서는 정규표현식이나 AspectJ의 문법을 이용하여 Pointcut 정의 가능 | sayHello() |
| Weaving | Advice(공통코드)를 핵심코드에 삽입<br>Spring은 Runtime시 Weaving을 지원 | HelloServiceProxy<br>클래스 생성 |
| Target<br>Object | 하나 또는 그 이상의 Aspect에 의해 Advice되는 객체<br>핵심 로직을 구현하는 클래스 | HelloService 객체 |


### 2-1-2. Weaving 방법
1. 컴파일 시 Weaving
  - 별도의 컴파일러를 통해 최종 바이너리가 만들어지는 방식
  - AspectJ프레임워크가 컴파일 시 Weaving 방법 사용
2. 클래스 로딩 시
  - 별도의 Agent를 이용하여 JVM이 클래ㅑ스를 로딩할 때 해당 클래스의 바이너리 정보 변경
  - AspectWerkz프레임위크가 클래스 로딩 시 Weaving 방법 사용
3. 런타임 시
  - 소스코드나 바이너리 파일의 변경 없이 Proxy를 이용하여 AOP를 지원하는 방식.
  - Spring AOP프레임워크는 Proxy를 이용한 런타임 시 Weaving하는 방법을 사용

### 2-1-3. Spring에서의 AOP
* Spring은 완전한 AOP기능을 제공하는 것이 목적이 아닌 애플리케이션을 구현하는데 필요한 기능을 제공하는 것을 목적으로 함.
* 필드값 변경 등 다양한 Jointpoint를 이용하려면 AspectJ등 다른 AOP솔루션을 이용해야함.
* Spring에서의 AOP구현 방법 3가지
  1. XML기반의 POJO 클래스를 이용한 AOP구현
  2. AspectJ에서 정의한 @Aspect Annotation기반의 AOP구현
  3. 스프링 API를 이용한 AOP 


<br><br><br>

## 2-2. XML을 이용한 AOP
### 2-2-1. AOP라이브러리 추가
* pom.xml에 dependency추가

```xml
  <!-- https://mvnrepository.com/artifact/org.aspectj/aspectjweaver -->
	<dependency>
	    <groupId>org.aspectj</groupId>
	    <artifactId>aspectjweaver</artifactId>
	    <version>1.9.7</version>
	   <!--  <scope>runtime</scope> -->
	</dependency>
	
	<!-- https://mvnrepository.com/artifact/org.aspectj/aspectjrt -->
	<dependency>
	    <groupId>org.aspectj</groupId>
	    <artifactId>aspectjrt</artifactId>
	    <version>1.9.7</version>
	    <!-- <scope>runtime</scope> -->
	</dependency>
```
<br>

### 2-2-2. aop 설정 추가.
* xml에서 beans의 xmlns:aop설정 추가

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd 
                      http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
	
</beans>
```

<br>

### 2-2-3. 클래스
* HelloController 클래스 : HelloService클래스를 상속받아 @Qualifier어노테이션으로 helloService 주입받음
* HelloService 클래스 : log()메소드 호출 X ( 핵심코드 )
* HelloLog 클래스 : log()메소드를 지닌 공통코드

<br>

### 2-2-4. AOP 설정
* 스프링 설정파일에서 Proxy객체를 생성하고, 핵심코드 생성시 공통코드가 실행되도록 설정
* application-config.xml 예시 코드

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:aop="http://www.springframework.org/schema/aop"
  xmlns:context="http://www.springframwork.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd 
                      http://www.springframwork.org/schema/context http://www.springframwork.org/schema/context/spring-context-4.3.xsd
                      http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
	
  <context:component-scan base-package="패키지명" />
  <bean id="helloLog" class="패키지명.HelloLog" />
  
  <aop:config>
    <aop:pointcut id="hello" expression="execution(* 패키지명..HelloService.sayHello(..))" />
    <aop:aspect ref="helloLog">
      <aop:before pointcut-ref="hello" method="log" />
    </aop:aspect>
  </aop:config>
 
</beans>
```

<br><br>

#### 01. \<aop:config>
* 최상위 태그
* 여러번 사용가능
* `<aop:pointcut>`, `<aop:aspect>`, `<aop:advisor>` 태그 포함 가능

#### 02. \<aop:pointcut>
* `<aop:config>` 또는 `<aop:aspect>`안에 정의 가능
* 공통코드가 실행되어야할 핵심코드를 의미
* id속성과 expression 속성을 지님
  - id : 각 pointcut의 고유 ID 값
  - expression : 공통코드가 실행되어야할 핵심코드를 지정. 별도 표현식이 존재함.
* pointcut은 JoinPoint의 부분 집합
* Advise가 적용되는 Joinpoint를 의미
* Spring에서는 정규표현식이나 AspectJ문법을 이용하여 Pointcut을 정의할 수 있음

##### 02-1. Pointcut 표현식 종류
1. bean : bean의 이름을 이용하여 pointcut 지정
2. within : 지정한 패키지 또는 클래스 내의 모든 메소드를 pointcut으로 지정
3. execution : 가장 일반적인 포인트컷 표현식. 메서드 타입별로 pointcut 지정가능
  - `*` : 모든 값
  - `..` : 0개 이상
  - execution(
    + `modifiers-pattern?` : 수식어 (public, protect ...) 생략가능
    + `ret-type-pattern` : 리턴타입 주로 `*`를 사용하여 모든 리턴타입 표현
    + `declaring-type-pattern`? : 클래스 이름 표기 (생략가능)<br>패키지 이름 뒤에 `..`를 쓰면 서브패키지도 찾음. 같은 패키지일 경우 패키지 이름 생략가능.<br>클래스 이름 뒤에 `+`를 붙이면 파생클래스(자식 클래스) 에서도 찾는다.
    + `name-pattern` : 메소드 이름 표기<br>`*`를 이용하여 메소드 전부 또는 일부 이름으로 사용 가능
    + `(param-pattern)` : 매칭될 파라미터 표기<br>`()`는 파라미터를사용하지 않는 메소드를 찾음<br>`(..)`는 파라미터가 0개 이상의 메소드를 찾음<br>`(*)`는 파라미터를 1개 포함.<br>`(*, *)`는 파라미터 2개 포함 (Integer, ..) Integer 파라미터 타입 설정도 가능
    + `throws-pattern?`) : 예외를 throws 하는 클래스의 이름을 지정.
  - 논리연산자 and, or, not 등의 사용이 가능함.
    + xml 문서에서 사용시 `&amp;&amp;`를 이용하거나 `and`문자열을 이용하여 표현
  - 예시
    + excution(\* kr.ac.kopo.service.EmployeeService.\*(..)) : kr.ac.kopo.service.EmployeeService클래스의 모든 메소드를 포인트 컷(핵심코드)으로 지정
    + execution(\* EmployeeService.\*(..)) : 같은 패키지 내의 EmployeeService 클래스의 모든 메소드를 포인트 컷으로 지정
    + execution(public \* EmployeeService.\*(..)) : EmployeeService클래스의 메소드 중 public인 모든 메소드를 포인트 컷으로 지정
    + execution(public EmpVO EmployeeService.\*(..)) : EmployeeService클래스의 메소드 중 public이고 리턴타입이 EmpVO인 메소드를 포인트 컷으로 지정
    + execution(public EmpVO EmployeeService.\*(EmpVO, ..)) : EmployeeService클래스의 메소드 중 public이고 리턴타입이 EmpVO인 메소드이며, 첫번째 파라미터가 EmpVO인 메소드를 포인트 컷으로 지정
    + execution(\* kr.ac..\*.\*Service.\*(..) \|\| \* kr.ac..\*.\*Repository.\*(..)) : Service 클래스와 Repository 클래스의 모든 메소드를 포인트 컷으로 

<br>

#### 03. \<aop:aspect>
* 공통코드 객체 지정.
* Advice와 pointcut 연결
* id, order, ref 속성을 지님
  - id : aspect 고유id 값
  - order : 특정 pointcut에 여러 개 Advice가 실행될 때 aspect의 순서를 제어
  - ref : 공통 bean(Aspect)의 이름 지정. 
* `<aop:pointcut>`태그와 `<aop:after>`, `<aop:after-returning>`, `<aop:after-throwing>`, `<aop:around>`, `<aop:before>` 등 Adivce 태그를 가질 수 있음

<br>

### 2-2-5. Advice
* 종류
  - before
  - after
  - after-returning
  - after-throwing
  - around

* Advice 설정 태그들은 method, pointcut, pointcut-ref, arg-names 속성을 지님
  - `method` : aspect 객체의 메서드 이름을 지정 ( 공통 코드를 의미 )
  - `pointcut` : pointcut 표현식을 지정하기 위한 속성
  - `pointcut-ref` : \<aop:pointcut\> 태그에 의해 미리 정의된 pointcut의 ID 지정
  - `arg-names` : pointcut 매개변수에 일치시킬 Advice 메서드 인지 이름 리스트를 (,)로 분리하여 나열

#### 01. \<aop:before>
* 핵심코드 실행 전에 공통코드 실행

```xml
<aop:before pointcut-ref="hello" method="log"/>
```

#### 02. \<aop:after>
* 핵심코드 실행 후 (return 값이 없을 경우 또는 finally 블록 실행 후) 공통코드 실행
* method, pointcut, pointcut-ref, arg-names 속성을 지님

```xml
<aop:after pointcut-ref="hello" method="log" />
```

#### 03. \<aop:after-returning>
* 핵심코드 메소드가 리턴한 다음 공통코드 실행
* returning 속성을 지님 : 리턴값이 전달 될 공통코드의 메서드 파라미터 이름 지정
  - <span style="color:blue">public void resultLog(Object resultObj) { ... }</span> 로 선언되었을 경우 아래와 같이 속성 추가

```xml
<aop:after-returning pointcut-ref="hello" method="resultLog" returning="resultObj" />
```

#### 04. \<aop:after-throwing>
* 핵심코드에서 예외 발생시 공통코드 실행
* throwing 속성을 지님 : 예외를 전달할 공통코드의 메소드 파라미터 이름 지정
  - <span style="color:blue">public void exceptionLog(Exception ex) { ... }</span> 로 선언되었을 경우 아래와 같이 속성 추가

```xml
<aop:after-throwing pointcut-ref="hello" method="exceptionLog" throwing="ex" />
```

#### 05. \<aop:around>
* 핵심코드가 실행되는 동안 공통코드 실행
* before, after, after-returning, after-throwing 을 한번에 해결할 수 있음
* around에 지정할 공통코드 메소드는 반드시 ProceedingJoinPoint 변수를 파라미터로 선언해야함
* 실행 시간을 체크하는 공통코드(Aspect) : PerformanceAspect.java

```java
public class PerformanceAspect{

  public Object trace(ProceedingJoinPoint joinPoint) throws Throwable{
    Signature s = joinPoint.getSignature(); // 핵심코드의 메소드 원형 리턴
    String methodName = s.getName();
    System.out.println("[Log]Before: " + methodName + "time check start");
    long startTime = System.nanoTime();
    
    Object result = null;
    try{
      result = joinPoint.proceed();
    }catch(Exception e){
      System.out.println("[Log]Exception: " + methodName);
    }finally{
      System.out.println("[Log]Finally: " + methodName);
    }
    
    long endTIme = System.nanoTime();
    System.out.println("[Log]After: " + methodName + " time check end");
    System.out.println("[Log] " + methodName + " Processing time is " + (endTime - startTime) + "ns");
    
    return result;
  }
}
```

<br>

* 공통코드를 실행시키기 위한 Spring 설정 파일

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:aop="http://www.springframework.org/schema/aop"
  xmlns:context="http://www.springframwork.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd 
                      http://www.springframwork.org/schema/context http://www.springframwork.org/schema/context/spring-context-4.3.xsd
                      http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
	
  <context:component-scan base-package="패키지명" />
  <bean id="helloLog" class="패키지명.HelloLog" />
  <bean id="performanceAspect" class="패키지명.PerformanceAspect" />
  
  
  <aop:config>
    <aop:pointcut id="hello" expression="execution(* 패키지명..HelloService.sayHello(..))" />
    
    <aop:aspect ref="helloLog">
      <aop:before pointcut-ref="hello" method="log" />
    </aop:aspect>
    
    <!-- PerformanceAspect.java 공통코드 주입 -->
    <aop:aspect ref="performanceAspect">
      <aop:around pointcut-ref="hello" method="trace" />
    </aop:aspect>
  </aop:config>
 
</beans>
```

#### 06. \<aop:advisor>
* 별도의 advise를 만들었을 경우 사용
* 트랜잭션 처리를 위한 어드바이스 `<tx:advice>`
* 트랜잭션 처리 대상을 pointcut으로 지정한 후 이를 묶어주는 용도로 사용

```xml
<aop:config>
  <aop:pointcut id="txPointcut" expression="execution(* kr.ac.kopo..*Service.*(..))"/>
  <aop:advisor advice-ref="txAdvice" pointcut-ref="txPointcut" />
</aop:config>
```

<br><br><br>

## 2-3. Annotation을 이용한 AOP
### 2-3-1. AOP 라이브러리 의존성 추가.
* pom.xml에 dependency추가

```xml
  <!-- https://mvnrepository.com/artifact/org.aspectj/aspectjweaver -->
	<dependency>
	    <groupId>org.aspectj</groupId>
	    <artifactId>aspectjweaver</artifactId>
	    <version>1.9.7</version>
	   <!--  <scope>runtime</scope> -->
	</dependency>
	
	<!-- https://mvnrepository.com/artifact/org.aspectj/aspectjrt -->
	<dependency>
	    <groupId>org.aspectj</groupId>
	    <artifactId>aspectjrt</artifactId>
	    <version>1.9.7</version>
	    <!-- <scope>runtime</scope> -->
	</dependency>
```

<br>

### 2-3-2. AOP Annotation 사용 설정
* spring xml 설정파일에 `<aop:aspectj-autoproxy>`태그 추가

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:aop="http://www.springframework.org/schema/aop"
  xmlns:context="http://www.springframwork.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd 
                      http://www.springframwork.org/schema/context http://www.springframwork.org/schema/context/spring-context-4.3.xsd
                      http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
  
  <context:component-scan base-package-"kr.ac.kopo" />
  <aop:aspectj-autoproxy/>
  
</beans>
```

<br>

### 2-3-3. AOP Annotation
#### 01. @Aspect
* 공통코드를 정의한 클래스 
* 공통코드 객체가 bean 으로 등록되어야 하므로 `@Component`와 함께 사용.
* `<context:component-scan base-package-"kr.ac.kopo" />` 태그가 추가되어 있어야 Component에 의해 bean으로 등록될 수 있음


#### 02. @Pointcut
* Aspect 클래스 내에 아무 기능도 구현하지 않은 메소드를 추가하고 그 위에 `@Pointcut`을 이용하여 포인트컷 표현식 작성.
* 공통코드가 실행되어야할 핵심코드 지정

```java
@Pointcut(value="execution(* kr.ac.kopo..*.sayHello(..))")
private void helloPointcut() { }
```

#### 03. 어드바이스
##### @Before
* 핵심코드 실행 전에 공통코드 실행

##### @After
* 핵심코드 실행 후 (return 값이 없을 경우 또는 finally 블록 실행 후) 공통코드 실행
* 예외가 발생해도 실행됨.


##### @AfterReturning
* 핵심코드 메소드가 리턴한 다음 공통코드 실행

##### @AfterThrowing
* 핵심코드에서 예외 발생시 공통코드 실행

##### @Around
* 핵심코드가 실행되는 동안 공통코드 실행









<br><br><br><br>
