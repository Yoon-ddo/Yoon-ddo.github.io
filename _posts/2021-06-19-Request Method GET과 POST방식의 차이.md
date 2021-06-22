---
title: Request Method GET과 POST방식의 차이
toc: true
toc_sticky: true
toc_label: "Contents of Page"

categories:
  - Study
tags:
  - Servlet_Study
---

<br><br>

# Request(요청)
* uri요청 방식에는 2가지가 있다.
   - GET
   - POST
* 요청방식은 <form>태그에 정의할 수 있는데, <form method="get/post"> 중 하나를 입력하면 된다.


<br><br><br><br>

# GET
* get방식으로 서버에 요청할 경우
* URL뒤에 ** ?id=aaa&pwd=bbb ** 이런식으로 입력한 내용이 뒤에 붙게 된다.
* 이는 보안에 취약한 방법이기 때문에 대부분의 웹서비스는 get방식을 잘 사용하지 않는다.

<br><br><br><br>

# POST
* post방식은 URL뒤에 값이 붙어나오는 것과는 달리 body태그에 값이 붙게 된다.
* 그래서 대부분의 웹서비스는 POST형식을 사용한다.
* 하지만, 값을 제대로 추출하기 위해서는 한 줄의 인코딩 코드가 필요하다.

```java

response.getCharacterEncoding("utf-8");

```

* 위의 코드를 service 하는 메소드 부분 처음에 작성해주어야 한글이 깨지지 않는다.

<br><br><br><br>
  
