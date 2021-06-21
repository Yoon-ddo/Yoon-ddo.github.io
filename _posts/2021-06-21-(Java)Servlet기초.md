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
* **Client ---( Request : 요청 )--> WAS**
  - request방식 : GET/POST
* **Client <--( Response : 응답 )-- WAS**
* client는 WAS에 요청하고, WAS는 client에 응답한다!
<br><br><br><br>

### 디렉토리 구조
![Servletdir구조](/assets/imgss/20210618-디렉토리구조.jpg)



