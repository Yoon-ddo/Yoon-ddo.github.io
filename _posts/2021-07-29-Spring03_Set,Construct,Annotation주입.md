---
title: Spring03_Set,Construct,Annotation주입
toc: true
toc_sticky: true
toc_label: "Contents of Page"
categories:
  - Study
tags:
  - Spring_Study
---

<br><br>

# 1. Setter주입과 Constructor주입
* setter주입은 매개변수가 1개, Constructor는 매개변수가 1개이상이다.
* xml파일에 등록

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
	
  <bean>
    
  </bean>
  
  
</beans>
```


## 1-1. Setter주입
* set메소드를 이용한 주입
* `property` 태그에 name, ref 속성 사용.
  - name : set메소드의 이름 (setId()라면 name="id"이다)
  - ref : 설정할 bean id
  - value : 값 설정 

### 1-1-1. collection도 쓸 수 있음.(List, Map, Set)
* List , Set

```xml
<bean class="di.collection.Test" id="test>
   <property name="list">
       <list>
            <value>aaa</value>
            <value>bbb</value>
            <value>ccc</value>
       </list>
  </property>
</bean>
```

* Map

```xml
<bean class="di.collection.Test" id="test>
   <property name="map">
       <map>
           <entry>
                <key>park</key>
                <value>박길동</value>
           </entry>
           <entry>
                <key>hong</key>
                <value>홍길동</value>
           </entry>
      </map>
  </property>
</bean>
```

### 1-1-2. 예제
* di.xml01.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
	
	
	
	<bean class="di.xml01.Car" id="car"/>
	<bean class="di.xml01.HankookTire" id="tire"/>
	
	
	<bean class="di.xml01.HankookTire" id="hankook"/>
	<bean class="di.xml01.KumhoTire" id="kumho"/>
	
	
	<bean class="di.xml01.Car" id="car2">
		<!-- name : setter주입하려는 대상 (setTire의 tire임) -->
		<property name="tire" ref="kumho"/>
<!-- 		<property name="tire" ref="hankook"/> -->
	</bean>
</beans>
```

<br><br>

## 1-2. Constructor
* 생성자를 이용한 주입
* `constructor-arg`태그에 속성을 붙여 사용
  - ref : 이미 만들어진 객체 삽입
  - value : 상수값 삽입
    + \<constructor-arg value="hello"/>와 \<value>hello\</value>는 같다.
  - index : 순서 제어
  - type : java.lang.String처럼 type 지정가능
    + 사용 예) value="10"일때 문자인지 숫자인지 구분해주고 싶을 때.  
* ref와 value는 바꿔써도 알아서 타입매칭해주지만 위험한 방법이다.

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
	
	<bean class="di.xml02.HankookTire" id="hankook" />
	<bean class="di.xml02.KumhoTire" id="kumho" />
	<bean class="di.xml02.Car" id="car">
		<!--  생성자 주입을 하겠다는 의미  -->
		<constructor-arg ref="hankook" />
	</bean>
	
	<bean class="di.xml02.Car" id="car2">
		<!-- new Car(new HankookTire(), new KumhoTire()) -->
		<constructor-arg ref="hankook"/>
		<constructor-arg ref="kumho"/>
	</bean>
	
	<bean class="di.xml02.Car" id="car3">
		<!-- new Car(Tire, String)  
		- ref : 이미만들어진 객체삽입, value : 상수값 삽입 
		  바꿔써도 알아서 타입매칭해주지만 위험한 방법이다. 
		  index 속성으로 순서를 제어할수도 있다.-->
		<constructor-arg ref="kumho" index="0"/>
		<constructor-arg value="hello" index="1"/>
	</bean>
</beans>
```

### 1-2-1. 예제

```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
	
	<bean class="di.xml02.HankookTire" id="hankook" />
	<bean class="di.xml02.KumhoTire" id="kumho" />
	<bean class="di.xml02.Car" id="car">
		<!--  생성자 주입을 하겠다는 의미  -->
		<constructor-arg ref="hankook" />
	</bean>
	
	<bean class="di.xml02.Car" id="car2">
		<!-- new Car(new HankookTire(), new KumhoTire()) -->
		<constructor-arg ref="hankook"/>
		<constructor-arg ref="kumho"/>
	</bean>
	
	<bean class="di.xml02.Car" id="car3">
		<!-- new Car(Tire, String)  
		- ref : 이미만들어진 객체삽입, value : 상수값 삽입 
		  바꿔써도 알아서 타입매칭해주지만 위험한 방법이다. 
		  index 속성으로 순서를 제어할수도 있다.-->
		<constructor-arg ref="kumho" index="0"/>
		<constructor-arg value="hello" index="1"/>
	</bean>	
</beans>
```

<br><br><br>

# 2. Annotation주입
* 자동주입
* Xml파일이 너무 커지는 것을 방지
* 자동주입 기능 사용시 스프링이 알아서 의존객체를 찾아서 주입
* beans태그에 설정값 필요 `xmlns:context`

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
                      http://www.springframework.org/schema/beans/spring-beans.xsd
                      http://www.springframework.org/schema/context
                      http://www.springframework.org/schema/context/spring-context.xsd">

  <context:annotation-config />
  
</beans>
```

* `<context:annotation-config />` 태그 추가
* Java파일에 의존주입 대상에 @Autowired (spring에서만 사용) 또는 @Resource 설정
* beans에 xmlns:context경로도 적어주어야함.
* 또는 context태그 이용

```xml
<context xmlns="http://www.springframework.org/schema/beans"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
                      http://www.springframework.org/schema/beans/spring-beans.xsd
                      http://www.springframework.org/schema/context
                      http://www.springframework.org/schema/context/spring-context.xsd">

  <beans:bean></bean>
  <annotation-config />
  
</context>
```

## 2-0. 어노테이션 종류와 차이
* Autowired
   - Type매칭 후 name매칭
* Resource
  - name매칭 후 Type매칭
* Inject
  - Type매칭 후 name매칭
* Resource와 Inject는 순수 자바 어노테이션
* Autowired와 Resource
  - 둘다 타입, 이름 , @Qualifier를 사용
  - Autowired는 required를 사용해서 필수여부 지정.
    + (required=false) : 의존주입하려는 객체가 없다면 Null로 비워두기
  
<br><br>

## 2-1. Autowired
* xml설정 : `<context:annotaion-config />`
*  Java설정
  - 변수설정 ( 멤버변수에도 설정가능 ) : 알아서 멤버변수에 주입
  - 생성자설정 : 생성자 호출되면서 자동으로 tire 주입
  - set메서드 설정 : 반드시 기본생성자를 가지고 있어야함!
* 자동주입 객체 찾을 때 `Type`매칭이 우선임.

### 2-1-1. 예제1 : Tire객체가 하나일때

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
	
	<context:annotation-config />
	
	<bean class="di.anno01.HankookTire" id="tire"/>
	<bean class="di.anno01.Car" id="car"/>

</beans>

```

```java
package di.anno01;

import org.springframework.beans.factory.annotation.Autowired;

public class Car {
	
	@Autowired // Car생성시 의존성을 지닌 Tire에 자동으로 객체 삽입
	private Tire tire;
	// Car의 (tire 변수) ->  HankookTire

	public Car() {
		System.out.println("Car()...");
	}

	//@Autowired // Autowired로 인해 생성자 호출되면서 자동으로 tire 주입
	public Car(Tire tire) {
		this.tire = tire;
		System.out.println("Car(Tire)...");
	}
	
	//@Autowired //반드시 기본생성자가 있어야함!!!!
	public void setTire(Tire tire) {
		this.tire = tire;
		System.out.println("setTire()...");
	}
	
	
	public void prnTireBrand() {
		System.out.println("장착된 타이어 : " + tire.getBrand() );
	}
}
```

```java
package di.anno01;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.GenericXmlApplicationContext;

public class DriverMain {

	
	public static void main(String[] args) {
		
		ApplicationContext context = new GenericXmlApplicationContext("di-anno01.xml");
		Car car = context.getBean("car", Car.class);
		car.prnTireBrand();
	}
}
```

### 2-1-2. Tire객체가 여러개일때
* Type매칭이 우선이다!
* 그냥 선언만해서 main에서 불러오면 single matching bean but found2: hankook, kumho 에러남.
* id가 Car클래스의 멤버변수와 이름이 같다면 같은게 주입됨.
  - 그러나 해당 Tire가 상속관계가 아니라면 id이름에 상관없이 상속받는 Tire가 들어감 

<br><br>

## 2-2. Resource
* Jar파일 필요 : `pom.xml파일` dependencies안에 아래 태그 복붙
* Resource는 이름(id)매칭먼저함

```xml
<!-- https://mvnrepository.com/artifact/javax.annotation/javax.annotation-api -->
<dependency>
    <groupId>javax.annotation</groupId>
    <artifactId>javax.annotation-api</artifactId>
    <version>1.3.2</version>
</dependency>
```

* Java설정
  - 변수
  - Set메소드 

* 생성자에는 붙일 수 없다.
* 적용순서
  1. name속성에 지정한 bean객체 찾기 (있으면 주입)
  2. name속성이 없을 경우 동일한 type의 bean객체를 찾는다.
  3. name속성이 없고, 동일한 type을 갖는 bean객체가 2개 이상일 경우 같은 이름을 가진 bean객체를 찾는다.
  4. name속성이 없고, 동일한 type을 갖는 bean객체가 2개 이상이고, 같은 이름을 가진 bean객체가 없는 경우 @Qualifier를 이용해서 주입할 bean객체를 찾는다.

### 2-2-1. 예제
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
	
	<context:annotation-config />
	<bean class="di.anno02.HankookTire" id="hankook" />
	<!-- <bean class="di.anno02.KumhoTire" id="tire" /> -->
	<bean class="di.anno02.NexenTire" id="tire" />
	<bean class="di.anno02.Car" id="car"/>
</beans>
```

<br><br><br>

## 2-3. Component-scan
* xml파일 설정을 통해 자동으로 빈으로 사용될 객체를 등록.
* 지정된 패키지 하위의 모든 패키지를 스캔하여 빈으로 등록
* 빈으로 등록되려면 자바 클래스에서 어노테이션을 사용해야함
* @Component(나머지를 포괄적으로 담고있음)
* @Controller(ds에서 날아오는 요청 처리)
* @Service(DB에서 받아온 데이터 처리 비즈니스 로직)
* @Repository(DB 접근)

```xml
<context:component-scan base-package="kr.co.mlec" />
```

* 설정값이 없는 경우 클래스 이름의 첫자를 소문자로 적용한 빈으로 등록

```java
// Class 이름이 ColaDrink일때

@Component //<bean class="xxx.ColaDrink" id="colaDrink" />
@Component("ham") //<bean class="xxx.ColaDrink" id="ham" />
```

<br><br>

## 2-4. xml등록하는것 Java로 만들기
* Config클래스 만들기
  - @Configuration
  - @Bean : 반드시 붙여주어야 만들어짐

```java
package di.java;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class Config {
	
	
	@Bean
	public Car car() {
		return new Car();
	}
	
	@Bean
	public HankookTire hankookTire() {
		return new HankookTire();
	}
	
	//@Bean // 반드시 붙여주어야 만들어짐
	public KumhoTire kumhoTire() {
		return new KumhoTire();
	}
}
```

* Car.java

```java
package di.java;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Component;



@Component
public class Car {
	
	@Autowired
	@Qualifier("kumho")
	private Tire tire;

	public Car() {
		System.out.println("Car()...");
	}

	public Car(Tire tire) {
		this.tire = tire;
		System.out.println("Car(tire)...");
	}
	
	public void setTire(Tire tire) {
		this.tire = tire;
		System.out.println("setTire()...");
	}
	
	public void prnTireBrand() {
		System.out.println("장착된 타이어 : " + tire.getBrand() );
	}

}
```

* DriverMain.java
  - Config에 정의된 Bean객체를 불러옴 

```java
package di.java;

import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class DriverMain {

	public static void main(String[] args) {
		
		ApplicationContext context = new AnnotationConfigApplicationContext(Config.class);
		
		Car car = context.getBean("car", Car.class);
		car.prnTireBrand();

	}

}
```

<br><br><br><br>



