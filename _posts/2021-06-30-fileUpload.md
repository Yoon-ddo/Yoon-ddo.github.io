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

# 응용 : 게시물에 파일 업로드하기.
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
create sequence seq_t_board_file_no nocache;

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

## 5. 첨부파일 저장
* write.jsp

```java

<%
  // Enumeration 타입
  Enumeration files = multiRequest.getFileNames(); //첨부한 파일 이름들.
    while(files.hasMoreElements()){
      String fileName = (String)files.nextElement();

      File f = multiRequest.getFile(fileName);

      // 첨부파일이 1개만 있을수도 있으니! if문
      if( f != null){
      
        //원본파일 이름 얻기
        String fileOriName = multiRequest.getOriginalFileName(fileName);
        
        //변형된 파일이름 얻기
        String fileSaveName = multiRequest.getFilesystemName(fileName);
        //System.out.println(fileOriName + " : " +fileSaveName);
        
        //파일사이즈 얻기 : 파일사이즈는 long형이다.
			  int fileSize = (int)f.length();
        
        // VO에 값 넣기
        BoardFileVO fileVO = new BoardFileVO();
        fileVO.setFileOriName(fileOriName);
        fileVO.setFileSaveName(fileSaveName);
        fileVO.setFileSize(fileSize);
        
        
      }
	}
%>

```

* 게시물을 저장하기 위한 시퀀스번호 추출해서 t_board, t_board_file DB에 넣어주어야함.
  - boardDAO.java 

```java

  /**
	 * t_board sequence추출
	 * @return
	 * @throws SQLException
	 */
	public int selectNo() throws SQLException  {
		
		int num = 0;
		
		StringBuilder sql = new StringBuilder();
		sql.append(" select seq_to_board_no.nextval from dual ");
		
		try(
				Connection conn = new ConnectionFactory().getConnection();
	 		 	PreparedStatement pstmt = conn.prepareStatement(sql.toString());	
			){
			
			ResultSet rs = pstmt.executeQuery();
			
			if(rs.next()) {
				num = rs.getInt(1);
			}
			
		}catch (Exception e) {
			e.printStackTrace();
		}
		
		return num;
	}

```

## 6. 파일정보 DB에 저장
* boardDAO.java

```java

  /**
	 * 업로드하는 파일정보
	 * @param fileVO
	 * @throws SQLException
	 */
	public void insertFile(BoardFileVO fileVO) throws SQLException{
		StringBuilder sql = new StringBuilder();
		sql.append("insert into t_board_file(no, board_no, file_ori_name, file_save_name, file_size) ");
		sql.append(" values(seq_t_board_file_no.nextval, ?, ?, ?, ?) ");
		
		try(
			Connection conn = new ConnectionFactory().getConnection();
 		 	PreparedStatement pstmt = conn.prepareStatement(sql.toString());
			){
			
			int loc = 1;
			pstmt.setInt(loc++, fileVO.getBoardNo());
			pstmt.setString(loc++, fileVO.getFileOriName());
			pstmt.setString(loc++, fileVO.getFileSaveName());
			pstmt.setInt(loc++, fileVO.getFileSize());
			
			pstmt.executeUpdate();
			
			
			
		}catch (Exception e) {
			e.printStackTrace();
		}
		
	}

```

## 7. 최종 write.jsp코드

```java

request.setCharacterEncoding("utf-8");
 
 	// 저장할 곳.
 	String saveDirectory = "D:/윤소영/web-workspace/Mission-Web/WebContent/upload";
  //얘는 개발자가보고있는 저장경로이고, 실제로 deploy할 는 eclipse-work폴더의 서버경로를 작성해야함!!!
  
  
 	MultipartRequest multiRequest = new MultipartRequest(request, saveDirectory, 1024*1024*3, "utf-8", new KopoFileNamePolicy());
 	
 	
 	//1. t_board 게시글저장
 	String title = multiRequest.getParameter("title");
 	String writer = multiRequest.getParameter("writer");
 	String content = multiRequest.getParameter("content");
 	
 	//게시물 번호 추출
 	boardDAO dao = new boardDAO();
 	int boardNo = dao.selectNo();
 	
 	BoardVO board = new BoardVO();
 	board.setTitle(title);
 	board.setWriter(writer);
 	board.setContent(content);
 	board.setNo(boardNo);
 	
 	
	dao.insert(board);	

	
	//2. t_board_file에 첨부파일 저장
	Enumeration files = multiRequest.getFileNames();
	while(files.hasMoreElements()){
		String fileName = (String)files.nextElement();
		
		File f = multiRequest.getFile(fileName);
		
		// 첨부파일이 1개만 있을수도 있으니! if문
		if( f != null){			
			//원본파일 이름 얻기
			String fileOriName = multiRequest.getOriginalFileName(fileName);
			//변형된 파일이름 얻기
			String fileSaveName = multiRequest.getFilesystemName(fileName);
			//System.out.println(fileOriName + " : " +fileSaveName);
			
			//파일사이즈 얻기
			int fileSize = (int)f.length();
			
			// VO에 값 넣기
			BoardFileVO fileVO = new BoardFileVO();
			fileVO.setFileOriName(fileOriName);
			fileVO.setFileSaveName(fileSaveName);
			fileVO.setFileSize(fileSize);
			
			//게시물을 저장하기 위한 시퀀스번호 추출해서 t_board, t_board_file DB에 넣어주어야함.
			fileVO.setBoardNo(boardNo);
			
			dao.insertFile(fileVO);
			
		}
	}
  
```

<br><br><br><br>

# 응용2 : 업로드한 파일 조회
## 1. 저장한 파일 출력할 부분 만들기
* detail.jsp

~~~html

<%
   //1. 게시물번호 추출
	int boardNo = Integer.parseInt(request.getParameter("no"));
	
	//2. 데이터베이스에서 조회
	boardDAO dao = new boardDAO();
	
	BoardVO board = dao.selectOne(boardNo);
	// 2-1. 조회수 증가
	dao.viewCnt(boardNo, board, 1);
	// 2-2. 게시물 조회
	board = dao.selectOne(boardNo);
	
	pageContext.setAttribute("board", board);
	
	// 2-3. t_board_file테이블에서 게시물의 첨부파일 조회
	List<BoardFileVO> fileList = dao.selectFileByNo(boardNo);
%>
   

  <tr>
    <th width="25%">첨부파일</th>
    <td></td>
  </tr>

~~~

<br>

## 2. 첨부한 파일 명단 저장
* boardDAO.java

```java

  /**
	 * 첨부한 파일 목록저장
	 * @param boardNo
	 * @return
	 * @throws SQLException
	 */
	public List<BoardFileVO> selectFileByNo(int boardNo) throws SQLException{
		
		List<BoardFileVO> list = new ArrayList<>();
		StringBuilder sql = new StringBuilder();
		sql.append(" select no, file_ori_name, file_save_name, file_size ");
		sql.append(" from t_board_file ");
		sql.append(" where board_no = ? ");
		
		try(
			Connection conn = new ConnectionFactory().getConnection();
			PreparedStatement pstmt = conn.prepareStatement(sql.toString());
			){
			
			pstmt.setInt(1, boardNo);
			
			ResultSet rs = pstmt.executeQuery();
			
			while(rs.next()) {
				
				BoardFileVO fileVO = new BoardFileVO();
				
				fileVO.setNo(rs.getInt("no"));
				fileVO.setFileOriName(rs.getString("file_ori_name"));
				fileVO.setFileSaveName(rs.getString("file_save_name"));
				fileVO.setFileSize(rs.getInt("file_size"));
				
				list.add(fileVO);
				
			}
			
		}catch (Exception e) {
			e.printStackTrace();
		}

		return list;
	}
 
```

<br>

## 3. 공유영역에 올리고, forEach태그로 출력
* detail.jsp

~~~html

<%
   
   //공유영역에 올리기
   // 2-3. t_board_file테이블에서 게시물의 첨부파일 조회
   List<BoardFileVO> fileList = dao.selectFileByNo(boardNo);

   pageContext.setAttribute("board",board);
   pageContext.setAttribute("fileList",fileList);


%>

<tr>
    <th width="25%">첨부파일</th>
    <td>
      <c:forEach items="${ fileList }" var="file">
      <a href="/Mission-Web/upload/${ file.fileSaveName }">
        <c:out value="${ file.fileOriName }" />
        <span style="color:lightgray">(${ file.fileSize } bytes)</span><br>
      </a>
      </c:forEach>

    </td>
</tr>


~~~

* a태그 href의 경로는 서버의 경로(eclipse-work폴더)를 작성해주어야 한다!!!!!

<br><br><br><br>










