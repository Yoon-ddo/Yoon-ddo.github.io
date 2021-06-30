---
title: fileUpload
toc: true
toc_sticky: true
toc_label: "Contents of Page"

categories:
  - Study
tags:
  - JSP_Study
---

<br><br>

# 파일 업로드
## 1. 파일 업로드 폼 생성
* fileUpload.jsp

~~~html

  <h2>파일 업로드 테스트</h2>
  <form action="uploadInfo.jsp" method="post">
    id : <input type="text" name="id"><br>
    file : <inpyt type="file" name="uploadfile"> <br>
    <input type="submit" value="전송">
  </form>

~~~

<br><br>

## 2. 업로드된 정보 출력
* uploadInfo.jsp
* 모든 서버와 클라이언트가 같은 언어로 만들어졌다는 보장이 없다.
* File IO 객체는 byte단위로 Stream한다.

```java

<%
	InputStreamReader isr = new InputStreamReader(request.getInputStream(), "utf-8");
	BufferedReader br = new BufferedReader(isr);
	
	while(true){
		String data = br.readLine();
		if(data == null){
			break;
		}else{
			out.println(data+"<br>");
		}
	}
%>

```

* id=&uploadfile=cat.jpg 로 출력
  - 실제 파일이 아닌 파일명만 전송됨.


### 해결방법1 : form태그에는 **enctype**속성이 있음

~~~html

  <h2>파일 업로드 테스트</h2>
	<form action="uploadInfo.jsp" method="post" enctype="multipart/form-data">
		id : <input type="text" name="id"><br>
		file : <input type="file" name="uploadfile"> <br>
		<input type="submit" value="전송">
	</form>

~~~

* 이렇게 전송된 정보는
* ------WebKitFormBoundary3twEt1nFA8B1dwP4<br>Content-Disposition: form-data; name="uploadfile"; filename="cat.jpg"<br>Content-Type: image/jpeg
* 의 정보가 출력된 후 한줄 띄우고 파일이 출력된다. 
* 하지만 내가 알던 사진파일이 아니네?!

### 해결방법2
1. servlets.com 접속
2. 왼쪽메뉴에 **COS File Upload Library** 클릭
3. Download에서 **cos-20.08.zip** 다운로드
4. 압축 풀고 lib/**cos.jar** 파일을 자바 프로젝트 WEB-INF/lib 폴더안에 넣는다.

* 예시 : multipartRequest.getParameter("title");



