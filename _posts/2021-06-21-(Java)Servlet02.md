---
title: (Java)Servlet02
categories:
  - Study
tags:
  - Java_Study
---

#### GET방식과 POST방식
* html <form>태그에 method로 GET/POST를 설정할 수 있고, 두개에는 차이가 있다.
  * **GET** : 폼에 입력 후, submit을 누르면 URL뒤에 **<span style="color:blue">?id=aaa</span>**이런식으로 내가 입력한 값이 URI뒤에 붙게된다.
  * **POST** : 반면 입력한 값이 URI뒤에 붙지 않고, 값이 body에 날라가서 보안성이 우수하다. 그래서 일반적으로 form태그에는 **<span style="color:blue">POST방식</span>**을 사용한다.
* 그런데 POST방식으로 id를 입력했더니 깨진다(ì¥ì¥)!??
    - POST방식은 body부분의 인자를 해석하기 위해 **반드시**인코딩이 필요하다.
    - service메소드에서 **<span style="color:red">request.setCharacterEncoding("utf-8");</span>** 로 인코딩을 해주어야한다.
<br>
  ```Java
     @Override
      protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        //body부분의 인자를 인코딩하기위해
        request.setCharacterEncoding("utf-8");


        StringBuffer url = request.getRequestURL();
        String method = request.getMethod();
        String uri = request.getRequestURI();
        String id = request.getParameter("id");
        System.out.println("method : "+method);
        System.out.println("uri : "+uri);
        System.out.println("url : "+url);
        System.out.println("id : " + id);

        response.setContentType("text/html; charset=utf-8");

        PrintWriter out = response.getWriter();
        out.println();
        out.println("<html>");
        out.println("	<head>");
        out.println("		<title>메소드 호출 방식 - POST</title>");
        out.println("	</head>");
        out.println("	<body>");
        out.println("=============================================<br>");
        out.println("<h1>요청결과</h1>");
        out.println("=============================================<br>");
        out.println("ID : " + id + "<br>");
        out.println("요청방식 : " + method + "<br>");
        out.println("URL : " + url + "<br>");
        out.println("URI : " + uri + "<br>");
        out.println("=============================================<br>");
        out.println("	</body>");
        out.println("</html>");

        out.flush();
        out.close();
      } 
  
  ```
<br><br><br>
  
  
#### GET방식과 POST방식  
  
