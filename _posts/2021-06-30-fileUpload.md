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

~~~html

<h2>파일 업로드 테스트</h2>
<form action="uploadInfo.jsp" method="post">
  id : <input type="text" name="id"><br>
  file : <inpyt type="file" name="uploadfile"> <br>
  <input type="submit" value="전송">
</form>

~~~

