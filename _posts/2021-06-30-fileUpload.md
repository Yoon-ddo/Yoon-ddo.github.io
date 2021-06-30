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
1. [servlets.com] 접속
2. 왼쪽메뉴에 **COS File Upload Library** 클릭
3. Download에서 **cos-20.08.zip** 다운로드
4. 압축 풀고 lib/**cos.jar** 파일을 자바 프로젝트 WEB-INF/lib 폴더안에 넣는다.

* MultipartRequest(javax.servlet.http.HttpServletRequest request, java.lang.String saveDirectory, int maxPostSize, FileRenamePolicy policy) 사용.
  - multipartRequest.getParameter("");
  - request, 저장경로, 파일사이즈, 파일저장명 규칙

<br><br><br><br>

# 게시물에 파일 업로드하기.
## 1. 파일업로드 폼 생성
* writeForm.jsp
* enctype="multipart/form-data"

~~~html

<form action="write.jsp" method="post" name="writeForm" onsubmit="return doWrite()" enctype="multipart/form-data">
  <input type="file" name="attachfile1"><br>
  <input type="file" name="attachfile2">
</form>

~~~

### 1-1. 파일이름 임의로 생성
* KopoFileNamePolicy.java

```java

public class KopoFileNamePolicy implements FileRenamePolicy {

	@Override
	public File rename(File f) {
		String name = f.getName();
		String ext = "";
		int dot = name.lastIndexOf(".");
		if (dot != -1) {
			ext = name.substring(dot); 
		} else {
			ext = "";
		}
		String str = "kopo" + UUID.randomUUID(); // 32비트의 임의의 수
		File renameFile = new File(f.getParent(), str + ext);
		return renameFile;
	}
}

```

<br>

## 2. write.jsp에 multipartRequest 객체 생성
* write.jsp

```java

<%
  request.setCharacterEncoding("utf-8");

    // 저장할 곳.
    String saveDirectory = "D:/윤소영/web-workspace/Mission-Web/WebContent/upload";

    MultipartRequest multiRequest = new MultipartRequest(request, saveDirectory, 1024*1024*3, "utf-8", new KopoFileNamePolicy());

%>
  
```

<br>

## 3. t_board_file 데이터베이스 생성
* t_board에 이름을 저장하게 되면 데이터베이스가 L자형태가 되기 때문에 따로 파일명만 저장하는 DB생성
* t_board_file
  - 파일번호
  - 게시물번호
  - 원본파일이름
  - 파일저장이름
  - 파일저장사이즈

```sql

-- 파일첨부
create table t_board_file(
    no number(5) primary key, -- 파일번호
    board_no number(5) not null,  --게시물 번호
    file_ori_name varchar2(300), --원본파일이름
    file_save_name varchar2(300), --파일저장이름
    file_size number(10),           -- 파일 저장사이즈
    constraint t_board_file_no_fk foreign key(board_no)
    references t_board(no) -- t_board의 게시물번호 참조
);

-- 시퀀스 생성
create sequence t_board_file_no nocache;

```

<br>

## 4. fileVO생성
* BoardFileVO.java

```java

  private int no;
	private int boardNo;
	private String fileOriName;
	private String fileSaveName;
	private int fileSize;

```











