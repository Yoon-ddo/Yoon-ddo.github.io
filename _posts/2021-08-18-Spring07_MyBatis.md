---
title: Spring07_MyBatis
toc: true
toc_sticky: true
toc_label: "Contents of Page"
categories:
  - Study
tags:
  - Spring_Study
---

<br><br>

# myBatis
* 기존에는 SQL 쿼리를 DAO에 직접 써서 DB에 접근했다.
* 항상 DB접근, PreparedState객체 등 같은 코드가 계속 앞뒤로 들어가게됨
* DB에서 쿼리만 제어해줄 수 있고, 날아오는 파라미터를 제어하는 Java로 만든 프레임워크
* 2010년 Apache(iBatis)에서 Google(myBatis)로 변경
* JDBC코딩을 위한 일반화된 프레임 워크
* SQL을 Java가 아닌 XML코드로 따로 분리
* SQL의 실행 결과를 Map 또는 VO로 자동 매핑
* SQL을 XML이나 인터페이스 내에 Annotation을 활용하여 처리
* 적용
  - SqlMapConfig파일(환경설정 파일)과 Mapper파일(실제 쿼리를 적용할 파일)이 필요( XML )
  - SqlSession객체를 얻어온 다음 myBatis와 연동

<br><br><br>

# 1. 환경설정(Console모드)
1. [MyBatis](https://mybatis.org/mybatis-3/ko/index.html)
2. [시작하기](https://mybatis.org/mybatis-3/ko/getting-started.html) 에서 jar파일(3.5.7) 다운로드
3. lib 폴더 만들어서 파일 복붙 후 BuildPath 라이브러리 추가
4. ojdbc8.jar 파일도 lib에 복붙 후 BuildPath 라이브러리 추가

<br><br><br>

# 2. XML을 이용한 방법
## mybatis-config.xml
* 환경설정파일

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <property name="driver" value="${driver}"/>
        <property name="url" value="${url}"/>
        <property name="username" value="${username}"/>
        <property name="password" value="${password}"/>
      </dataSource>
    </environment>
  </environments>
<!--   <mappers>
    <mapper resource="org/mybatis/example/BlogMapper.xml"/>
  </mappers> -->
</configuration>
```

* driver, url, username, password는 바뀔 수도 있는 값이다.
  - properties 파일에 넣어두고 그때그때 바꾸어준다.

* mybatic-config.xml파일에서 `typeAlias`를 지정하면 VO 객체를 간격하게 사용할 수 있다.

```xml
<typeAliases>
  <typeAlias  type="kr.ac.kopo.board.BoardVO" alias="boardVO"/>
</typeAliases>
```

<br>

## db.properties
* db접근 정보를 담은 파일

```
jdbc.driver=oracle.jdbc.driver.OracleDriver
jdbc.url=jdbc:oracle:thin:@localhost:1521:xe
jdbc.user=scott
jdbc.password=tiger
```

* 위와 같이 설정해주면 아래와 같이 `mybatis-config.xml` 를 수정한다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	<properties resource="db.properties" />
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <property name="driver" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
      </dataSource>
    </environment>
  </environments>
<!--   <mappers>
    <mapper resource="org/mybatis/example/BlogMapper.xml"/>
  </mappers> -->
</configuration>
```

## MyConfig.java
* MyConfig객체가 생성되면 xml 파일을 읽고 sqlSessionFactory와 SqlSession객체를 얻어옴
* DB접근

```java
package kr.ac.kopo;

import java.io.InputStream;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

public class MyConfig {
	
	private SqlSession session;
	
	public MyConfig() {
		// 객체 생성시 바로 실행되도록
		try {
			String resource = "mybatis-config.xml";
			InputStream inputStream = Resources.getResourceAsStream(resource);
			SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
			session = sqlSessionFactory.openSession();
			//System.out.println(session);
		
		
		}catch (Exception e) {
			e.printStackTrace();
		}
	}
	
	public SqlSession getInstance() {
		// 외부에서 접근할 수 있도록 session객체 얻어오는 메소드
		return session;
	}
}
```

## MyBatisMain.java
* MyConfig.java객체를 생성

```java
package kr.ac.kopo;

import org.apache.ibatis.session.SqlSession;

public class MyBatisMain {
	public static void main(String[] args) {
		SqlSession session = new MyConfig().getInstance();
		System.out.println(session);
	}
}
```


## mapper파일
* 실제 쿼리를 적용할 파일
* 태그종류
  - select
    + `resultType` : select했을 때 받아오는 값의 타입 지정.
  - insert
  - update
  - delete
  - resultMap

* 속성 종류
  - 구문을 구분하는 속성은 `id`속성이다.
  - `parameterType` : 객체로부터 날아온 객체의 type
    + parameterType="kr.ac.kopo.board.BoardVO"
    + 객체 링크확인

```xml
<select id="selectAll">
         select * 
         from t_board
</select>

<select id="selectByNo">
         select * 
         from t_board
         where no = 5
</select>
```

* mapper파일이 하나가 아닐 수도 있다.
  - `mapper1.xml`파일에 `id="selectAll"`이 있고 `mapper2.xml`에도 `id="selectAll"`이 있을수도 있음
  - namespace (package와 같은 역할) 태그를 붙여주면 구분할 수 있음
    + `<mapper namespace="member.dao">` : member.dao의 selectAll이라는 의미

<br>

* VO객체 변수이름과 컬럼명이 다를 때 맞춰주고 싶다면?
  - `resultMap` 태그
  - DAO에서 불러올 때는 `List<BoardVO> list = session.selectList("board.BoardDAO.selectAllMap");`로 불러온다.

```xml
<resultMap type="boardVO" id="boardMap">
  <result column="view_cnt" property="viewCnt"/>
  <result column="reg_date" property="regDate"/>
</resultMap>

<select id="selectAllMap" resultMap="boardMap">
  <!-- select no, title, writer, content, view_cnt, to_char(reg_date,'yyyy-mm-dd') as reg_date -->
  select no, title, writer, content, view_cnt , reg_date
  from t_board
  order by no desc
</select>
```



<br>

### board.xml
* common.db패키지 생성 후 board.xml 파일 생성 (mapper파일을 관리하는 패키지)
* sql문 뒤에 `;`는 쓰지 않는다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="board.BoardDAO">
<!-- kr.ac.kopo 생략된것. -->
	<insert id="newBoard">
		insert into t_board(no, title, writer, content)
			values(seq_tboard_no.nextval, 'mybatis연습', 'hong', 'insert')
	</insert>
</mapper>
```

<br>

* mybatis-config.xml 파일 mapper 태그 수정

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	<properties resource="db.properties" />
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <property name="driver" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
      </dataSource>
    </environment>
  </environments>
  <mappers>
    <mapper resource="common/db/board.xml"/>
  </mappers>
</configuration>
```

<br><br><br>

## DAO객체
* SqlSession객체가 있는 곳

### 1. BoardDAO.java에서 insert 실행


```java
package kr.ac.kopo.board;

import org.apache.ibatis.session.SqlSession;

import kr.ac.kopo.MyConfig;

public class BoardDAO {
	private SqlSession session;
	
	public BoardDAO() {
		// MyConfig객체로 DB접근
		session = new MyConfig().getInstance();
		System.out.println(session);
	}
	
	public void work() {
		insert();
	}
	
	private void insert() {
		
		BoardVO vo = new BoardVO();
		vo.setTitle("객체로 삽입");
		vo.setWriter("홍길동");
		vo.setContent("삽입왜안돼 ㅠ");
		
		// "namespace.id"형식으로 작성
		session.insert("board.BoardDAO.newBoard", vo);
		// commit필요
		session.commit();
		
		System.out.println("삽입완료");
	}

}
```

<br><br>

### 2. BoardDAO.java에서 selectAll 실행
* board.xml에 추가
  - resultType을 boardVO로 해주었기 때문에 알아서 VO객체로 받아옴.
  - `session.selectList()`알아서 List형태로 가져옴
 
```xml
<select id="selectAll" resultType="boardVO">
  select * 
  from t_board
  order by no desc
</select>
```

<br>

* BoardDAO.java 수정

```java
public void work() {
	
		//insert();
		selectAll();
	}
	
	private void selectAll() {
		//board.xml파일에서 return타입설정해줬기 때문에 자동으로 형변환 됨.
		List<BoardVO> list = session.selectList("board.BoardDAO.selectAll");
		for(BoardVO board:list) {
			System.out.println(board);
		}
	}
```

<br><br>

### 3. BoardDAO.java에서 selectOne 실행
* 하나의 데이터( ex. 번호로 조회 )를 가져오기 위해서는 List를 쓸 필요 없음.
* `session.selectOne();`는 알아서 Object객체로 가져옴

#### 3-1. 파라미터 타입이 VO 일때
* mapper파일
  - parameterType이 VO일때는 변수에 맞는 이름을 작성 : `#{no}`

```xml
<select id="selectOne" resultMap="boardMap" parameterType="boardVO">
  select no, title, writer, view_cnt, to_char(reg_date,'yyyy-mm-dd') reg_date
  from t_board
  where no = #{no}
</select>
```

<br>

* DAO
 
```java
private void selectOne() {
		BoardVO vo = new BoardVO();
		vo.setNo(42);
		BoardVO board = session.selectOne("board.BoardDAO.selectOne", vo);
		System.out.println(board);
}
```

<br><br>

#### 3-2. 파라미터 타입이 int 일때
* mapper파일

```xml
<select id="selectOne2" resultMap="boardMap" parameterType="int">
<!-- int타입일때는 무엇을 쓰든 상관 없음 -->
  select no, title, writer, view_cnt, to_char(reg_date,'yyyy-mm-dd') reg_date
  from t_board
  where no = #{no}
</select>
```

<br>

* DAO

```java
private void selectOne2() {
		BoardVO board = session.selectOne("board.BoardDAO.selectOne2", 42);
		System.out.println(board);
}
```

<br><br>

#### 3-3. 중복되는 코드 묶기
* `<sql>`태그와 `<include>`를 이용하여 중복되는 코드를 묶을 수 있다.
* selectOne과 selectAll에서 중복되는 공통의 쿼리를 묶어주는 역할

```xml
<sql id="selectBoard">
  select no, title, writer, view_cnt, to_char(reg_date,'yyyy-mm-dd') reg_date
  from t_board
</sql>


<select id="selectOne3" resultMap="boardMap" parameterType="int">
    <include refid="selectBoard"/>
  where no = #{no}
</select>
```

<br><br><br>

## 동적으로 수행하기
* 제목으로 검색, 작성자로 검색 기능을 구현할 때 sql에서 바뀌는 부분은 where절만 달라지게 됨
  - where title= ? 또는 where writer= ?
  - 조건에 따라 다른 쿼리를 작동하게 하는 것 : 동적태그

* 태그
  - `<foreach>` : where addr in ('seoul','busan','jeju')의 동작이 가능하게 해줌
    + `<foreach item="addr" open="(" seperator="," close=")" collection="addrs"></foreach>`
    + collection
    + item
    + open : 첫 시작을 괄호로 설정가능
    + seperator : 구분자
    + close : (괄호) 닫기
  - `<if>`
  - `<choose>`
  - `<where>`
    + `<where> <if test="writer != null"> writer like #{writer} </if> </where>` 
    + <span style="color:red">다른 조건에 AND가 붙어있어도 where을 사용하면 알아서 생략해준다</span>















<br><br><br><br>
