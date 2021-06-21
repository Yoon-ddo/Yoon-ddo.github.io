---
title: (Java)Servlet기초
categories:
  - Study
tags:
  - Java_Study
---


### Servlet이란?
* Client와 WAS서버에는 통로가 있는데, 이 통로를 통해 html형식의 데이터를 전송하는 기술이다.
* html을 jsp로 바꾸어도 실행할 수 있도록 해준다.
  - ASP보다 JSP가 보안이 더 좋다.
* 예전에는 Servlet이 불편해서 사용하는 사람이 별로 없었다고 한다.
* 하지만 웹의 규모가 커지면서 유지보수하기 좋고 수월하게 개발할 수 있게 Servlet + JSP(모델2방식)이 급부상하면서 사용량이 증가했다.
  - Servlet - 화면을 구성하지 않고 요청을 관리하고 비즈니스 로직을 관리한다.
  - 모델2방식 : MVC방식을 이용한다.
* 자바기반의 컴포넌트
* 변천사
  - html을 핸들링 하기 어려워서 JSP가 등장했고,
  - JSP에서 html의 코드가 복잡해 짐에 따라 Java와 html을 분리하였다.
  - 이때 MVC 패턴을 사용하여 해결
  - 모델 : JAVA(VO,DAO)
  - 뷰 : JSP
  - 컨트롤러 : Servlet

<br><br><br><br>

### Servlet API
* Servlet : 인터페이스
* Generic Servlet : 추상클래스
* HttpServlet : 추상클래스 **대부분 웹개발시 이를 상속한다**
* 웹에서 동작하는 서블릿 클래스가 되기위해서는 셋중 하나를 상속받아야한다! **extends/implements**

<br><br><br><br>


### Serlvet 프로세스 구조
#### Client ---( Request : 요청 )--> WAS
  - request방식 : GET/POST
#### Client <--( Response : 응답 )-- WAS
* client는 WAS에 요청하고, WAS는 client에 응답한다!

<br><br><br><br>

### Serlvet 디렉토리 구조
![Servletdir구조](/assets/imgss/20210618-디렉토리구조.jpg)
<br>
* 웹서비스를 할때 Java Resources/src/의 .java 파일들은 WEB-INF/classes에 있는 .class파일이 수행된다.
* 그러나 이는 보안폴더이므로 가상의 URL이 필요하다.
* 이를 **web.xml** 에 설정해주는 것이 바로 가상의 URL을 생성하는 과정!

<br><br><br><br>


### Serlvet 가상 URL 만들기

```Java
<display-name>Lecture-web</display-name>

  <servlet>
    <servlet-name>hello</servlet-name>
    <servlet-class>kr.ac.kopo.servlet.HelloServlet</servlet-class>  
  </servlet>
  <servlet-mapping>
      <servlet-name>hello</servlet-name>
      <url-pattern>/hello</url-pattern>
  </servlet-mapping>

<welcome-file-list>
 ```
 
* 기본적으로 **display-name**태그와 **welcom-file-list**태그 사이에 servlet을 구성한다.

###### <Servlet>
* **servlet-name**은 mapping태그의 servlet-name과 같아야하며, **servlet-class**태그에는 **패키지명과, java파일명**을 입력하면 된다.(확장자명은 입력X)
  
###### <Servlet-mapping>
*  **servlet-name**은Servlet태그의 servlet-name과 같아야하며, **url-pattern**은 root경로 다음에 붙을 이름을 지정해준다.
<br><br><br><br>

### Servlet 생명주기
* init : 최초 한번만 실행
* service : **필수** 요청에대한 응답 
  - 안만들면 405에러와 만나게된다 (url은 만들어졌는데 처리할 서비스가 없어 서블릿에러)
* destroy : 메모리 해제시 호출 
<br><br><br><br><br>


