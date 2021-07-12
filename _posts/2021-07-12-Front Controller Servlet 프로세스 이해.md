---
title: Front Controller Servlet 프로세스 이해
toc: true
toc_sticky: true
toc_label: "Contents of Page"

categories:
  - Study
tags:
  - Servlet_Study
---

<br><br>


# Servlet
## 1. FrontControllerServlet
- 모든 요청을 하나의 통로에서 처리하는 Servlet
- 어떤 URL이 들어와도 한 번에 처리하는 대표 Servlet
- FrontControllerServlet.java

```java
package kr.ac.kopo;

import java.io.IOException;

import javax.servlet.RequestDispatcher;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import kr.ac.kopo.controller.BoardListController;
import kr.ac.kopo.controller.WriteFormController;

public class FrontControllerServlet extends HttpServlet{
	
	
	
	public void service(HttpServletRequest request, HttpServletResponse response) throws IOException, ServletException{
		
		String context = request.getContextPath();
		//System.out.println("context : " + context); 
		// /Mission-Web-MVC01 자동으로 받아옴.
		
		String uri = request.getRequestURI();
		uri = uri.substring(context.length());
		//System.out.println("요청 URI : " + uri);
		
		String callPage = "";
		try {
		switch(uri) {
			case "/board/list.do":
				System.out.println("게시판 목록 처리");
				BoardListController list = new BoardListController();
				callPage = list.handleRequest(request,response);
				
				break;
				
				
			// "/Mission-Web-MVC01/board/writeForm.do":
			case "/board/writeForm.do":
				System.out.println("게시판 글등록 처리");
				WriteFormController writeForm = new WriteFormController();
				callPage = writeForm.handleRequest(request, response);
				
				break;
			}
		
		// sendRedirect하거나 forward
		// RequestDispatcher : java에서 forward
		RequestDispatcher dispatcher = request.getRequestDispatcher(callPage);
		dispatcher.forward(request, response);
		
			
		}catch (Exception e) {
			e.printStackTrace();
			// BoardListController에서 발생한 에러를 다시 service로 넘겨주는 것.
			throw new ServletException(e);
		}
	}
```

- web.xml

```xml
	<servlet>
	 	<servlet-name>frontController</servlet-name>
	 	<servlet-class>kr.ac.kopo.FrontControllerServlet</servlet-class>  
	</servlet>
	<servlet-mapping>
	  	<servlet-name>frontController</servlet-name>
	  	<url-pattern>*.do</url-pattern>
	</servlet-mapping>
```

<br><br>

## 2. Controller
- 각각의 요청을 처리
- .java 파일로 구성됨 Servlet 필요없음
- FrontController에서 req, res값을 넘겨주면 url을 받아오는 역할 수행
- BoardListContoller.java

```java
package kr.ac.kopo.controller;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class BoardListController {
	
	public String handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
		return "/board/list.jsp";
	}
}
```

- 코드 재사용성을 위해 Interface 생성.

```java
package kr.ac.kopo.controller;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public interface Controller {
	// 코드 재사용성을 위해 Interface생성.
	String handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception;

}
```

### HandlerMapping객체 이용
- 객체가 만들어지는 순간 구성이 되도록 생성자에 선언
- switch를 쓰지 않아도 된다.
- HandlerMapping.java

```java
public HandlerMapping() {
		Map<String, Controller> mappings = new HashMap<>();
		mappings.put("/board/list.do", new BoardListController());
		mappings.put("/board/writeForm.do",new WriteFormController());
    // uri가 들어오면 Controller실행 (FrontController.java에 구현 : Controller control = mappings.getController(uri); )
	}
  ```
 
 - 그런데 이렇게되면 service가 호출될때마다 계속 생기니까 성능저하가 일어나게됨
 - init메소드를 만들자!
 - FrontControllerServlet.java

```java
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import kr.ac.kopo.controller.Controller;
import kr.ac.kopo.controller.HandlerMapping;

public class FrontControllerServlet extends HttpServlet{
	
	private HandlerMapping mappings;
	
	@Override
	public void init(ServletConfig config) throws ServletException{
		mappings = new HandlerMapping();
	}
	
	@Override
	public void service(HttpServletRequest request, HttpServletResponse response) throws IOException, ServletException{
		
		String context = request.getContextPath();
		
		String uri = request.getRequestURI(); // 요청 URI분석
		uri = uri.substring(context.length());
		
		try {
    
			Controller control = mappings.getController(uri);
			
			if(control != null) {
      
				String callPage = control.handleRequest(request, response);
        
        // forward , callPage : forward시킬 주소.
				RequestDispatcher dispatcher = request.getRequestDispatcher(callPage);
				dispatcher.forward(request, response);			
				
			}
			
		}catch (Exception e) {
			e.printStackTrace();
			throw new ServletException(e);
		}
	}

}
```

- HandlerMapping은 Spring에서 블랙박스이므로 값을 받아올 방법을 고민해야함.

# Servlet Process
- Server와 DB 구성
  * Server
    + FrontControllerServlet
    + HandlerMapping객체
    + Controller(인터페이스)
    + Controller를 상속받는 여러개의 Controller
    + JSP
    + request공유영역
    + DB

![Servlet](/assets/imgss/20210712-ServletProcess.png)

<br>

1. Client 요청
2. FrontControllerServlet에 요청
3. HM에 uri전달
4. Controller 반환
5. Controller는 DB에 데이터 주고받고
6. request영역에 등록
  - 요청은 servlet이 받았고, 응답은 jsp가 하기때문에 모두가 참조할 수 있게 request영역에 올려야함
7. jsp주소를 Controller에서 FCS로 전달
8. FCS->JSP로 forward
9. JSP는 request공유영역을 보고 원하는 정보 구성
10. Client에 응답 

