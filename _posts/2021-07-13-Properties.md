---
title: Reflction과 Properties
toc: true
toc_sticky: true
toc_label: "Contents of Page"

categories:
  - Study
tags:
  - Servlet_Study
---

<br><br>


# Properties파일
- HandlerMapping과 FrontControlServlet을 건드리지 않고 uri를 바꿔주는 파일
- uri가 들어왔을 때 실제 삽입할 정보를 저장한 파일
- Web Project 아래에 bean.properties 파일 생성

```java
/board/list.do=kr.ac.kopo.controller.BoardListController
```

- Properties파일을 읽는 객체를 HandlerMapping에 생성

# Reflection
- 동적으로 파일을 읽고, `new ddd()`형식으로 객체를 만들 수 있다.
## 1. 예시

```java
try {

			//reflection
			Class<?> clz = Class.forName("kr.ac.kopo.controller.BoardListController"); // class정보 로딩
			BoardListController list = (BoardListController)clz.newInstance(); // 객체 생성
			// new BoardListController()와 같다.
			//<bean class="kr.ac.kopo.controller.BoardController" id="list" />
			System.out.println(list.handleRequest(null, null));

			// 클래스파일 경로를 알면 사용할 수 있음
			//Class<?> clz = Class.forName("java.util.Random");
			//java.util.Random r = (java.util.Random)clz.newInstance();
			//System.out.println("난수" + r.nextInt(10));
			
		}catch (Exception e) {
			e.printStackTrace();
		} 
```

## 2. 메소드 구현
- HandlerMapping.java

```java
public HandlerMapping() {

		/* 아래의 정보를 properties 파일에 저장하고 추출해보자.
		mappings = new HashMap<>();
		mappings.put("/board/list.do", new BoardListController());
		mappings.put("/board/writeForm.do",new WriteFormController());
		mappings.put("/board/detail.do", new BoardDetailController());
		mappings.put("/member/list.do", new MemberListController());
		*/
		
		mappings = new HashMap<>();
		Properties prop = new Properties();
		try {
			InputStream is = new FileInputStream("D:\\2021_java_db\\Web-Project\\Mission-Web-MVC01\\bean.properties");
      // 경로가 바뀔수도 있는데 절대경로를 쓴다구???
      
      
			prop.load(is);
			Set<Object> keys = prop.keySet();
			
			for(Object key :keys) {
				//System.out.println(key);//uri
				String className = prop.getProperty(key.toString());
				//System.out.println(className);
				Class<?> clz = Class.forName(className);
				mappings.put(key.toString(), (Controller)clz.newInstance());
				
			}
			
			
		}catch (Exception e) {
			e.printStackTrace();
		}
	}
  
 ```
 
 - InputStream is = new FileInputStream("D:\\2021_java_db\\Web-Project\\Mission-Web-MVC01\\bean.properties");
  * 경로가 바뀔수도 있는데 절대경로를 쓴다구???
- FrontControllerServlet.java파일의 init에서 인식할 수 있게 web.xml파일에 <init-param>태그로 정의한다.
- web.xml

```xml
<init-param>
  		<param-name>propertyLocation</param-name>
  		<param-value>D:\\2021_java_db\\Web-Project\\Mission-Web-MVC01\\bean.properties</param-value>
</init-param>
```
 
- FrontController.java
  
```java
@Override
	public void init(ServletConfig config) throws ServletException{
		
		//mappings = new HandlerMapping(); // Map이 한번만 만들어짐.
		// 무슨 controller쓸지 정보가 담겨있는애
		
		
		// bean.properties파일 경로를 init에서 인식할 수 있게 web.xml파일에 <init-param>태그로 정의한다.
		String propLoc = config.getInitParameter("propertyLocation"); 
		// propertyLocation이라는 이름을 지닌 파라미터의 값을 받는다.
		
		mappings = new HandlerMapping(propLoc);
	}
```
  
- HandlerMapping.java  

```java
  public HandlerMapping(String propLoc) {

		/* 아래의 정보를 properties 파일에 저장하고 추출해보자.
		mappings = new HashMap<>();
		mappings.put("/board/list.do", new BoardListController());
		mappings.put("/board/writeForm.do",new WriteFormController());
		mappings.put("/board/detail.do", new BoardDetailController());
		mappings.put("/member/list.do", new MemberListController());
		*/
		
		mappings = new HashMap<>();
		Properties prop = new Properties();
		try {
			InputStream is = new FileInputStream(propLoc);
			// 경로가 바뀔수도 있는데 절대경로를 쓴다구???
			
			prop.load(is);
			Set<Object> keys = prop.keySet();
			
			for(Object key :keys) {
				//System.out.println(key);//uri
				String className = prop.getProperty(key.toString());
				//System.out.println(className);
				Class<?> clz = Class.forName(className);
				mappings.put(key.toString(), (Controller)clz.newInstance());
				
			}
			
			
		}catch (Exception e) {
			e.printStackTrace();
		}
	}
```




<br><br><br>
  




