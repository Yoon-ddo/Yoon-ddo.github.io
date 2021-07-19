---
title: SpartaCoding_웹개발의 봄, Spring 2주차
toc: true
toc_sticky: true
toc_label: "Contents of Page"

categories:
  - Study
tags:
  - SpartaCoding_Study
---

<br><br>


# 1. RDBMS
- 컴퓨터에 정보를 저장하고 관리하는 기술


## 1-2. H2
* In-memory DB의 대표주자.
* 서버가 작동을 멈추면 데이터가 모두 삭제되는 DB

## 1-3. MySQL
* 스프링과 궁합이 좋음

<br><br><br>

# 2. H2 웹콘솔 설정
* IntelliJ 프로젝트에서 src > main > resources > application.properties 에 아래의 내용 복붙
  - spring.h2.console.enabled=true  
    spring.datasource.url=jdbc:h2:mem:testdb
* src > main > java > Week02Application 실행
* localhost:8080/h2-console 에서 Connect버튼 클릭
  - 에러시 JDBC URL칸에 jdbc:h2:mem:testdb  
