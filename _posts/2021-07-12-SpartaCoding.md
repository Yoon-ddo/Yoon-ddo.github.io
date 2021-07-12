---
title: SpartaCoding_웹개발의 봄, Spring 1주차
toc: true
toc_sticky: true
toc_label: "Contents of Page"

categories:
  - Study
tags:
  - SpartaCoding_Study
---

<br><br>


# 1. Settings
- JAVA 8 : 환경변수까지 설정할 것!
  * cmd 창에서 `java -version` 입력했을 때 버전이 나오면 성공한 것 
- IntelliJ IDEA
- ARC

<br><br><br>

# 2. Java기본문법
- JAVA 기본문법은 교육장에서 계속 해왔으니 Skip

<br><br><br>

# 3. 브라우저에 바로 나타내기
## 3-1. 화면에 클래스 정보 띄우기 전 기본개념
- Rest
  * 서버의 응답이 JSON 형식임을 나타냅니다.
  * HTML, CSS 등을 주고받을 때는 Rest 를 붙이지 않습니다.
- Controller
  * 자동응답기와 같다.
  * 말을 걸면 응답.
  * 클라이언트 요청(Request)을 받는 코드
- RestController
  * 요청받은 후, JSON만을 돌려주는 코드. 

## 3-2. RestController만들기
1. src > main > com.sparta.week01 에 controller 패키지를 만듭니다.
2. Course.java클래스파일 (Getter, Setter)

```java
package com.sparta.week01.prac;

public class Course {

    private String title;
    private String tutor;
    private int days;

    public Course(){

    }


    // Getter
    public String getTitle() {
        return this.title;
    }
    // Getter
    public String getTutor() {
        return this.tutor;
    }
    // Getter
    public int getDays() {
        return this.days;
    }

    // Setter
    public void setTitle(String title) {
        this.title = title;
    }
    // Setter
    public void setTutor(String tutor) {
        this.tutor = tutor;
    }
    // Setter
    public void setDays(int days) {
        this.days = days;
    }

}
```

3. CourseController.java 클래스파일 생성

```java
package com.sparta.week01.controller;

import com.sparta.week01.prac.Course;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class CourseController {

    @GetMapping("/courses")
    public Course getCourse(){
        Course course = new Course();
        course.setTitle("웹개발의 봄 스프링");
        course.setDays(35);
        course.setTutor("남병관");
        return course;
    }
}
```

- 입력 후, http:\/\/localhost:8080/courses 링크로 가면 아래의 결과가 나옴.

```java
{
title: "웹개발의 봄 스프링",
tutor: "남병관",
days: 35
}
```

#### 3-2-1. Spring이 하는 일?!
- 우리가 메소드만 잘 만들고, 매핑만 잘 해주면 원하는 요청이 들어왔을 때 스프링이 알아서 자동으로 처리해줌.
- JSON으로 바꿔주고 메소드 호출해주고 ... 등등 과정을 모두 스프링이 해준다.

<br><br><br><br>

# 4. Gradle이란?
- 많은 개발자들은 남의 코드에 의존한다.
- Library를 가져올때 쓴다.
- 배포할 때 쓴다.
## 4-1. 다른 사람들이 만들어둔 도구 내려받기
- JavaScript : NPM
- Python : pip
- Java : mavenCentral, jcenter

## 4-2. Maven Repository
1. ![Maven Repository](https://mvnrepository.com/)
2. `JSON In Java` 검색 후 20160810 버전 클릭
3. Gradle 누르고 복사
4. IJ에서 `build.gradle`파일에 `dependencies` 안에 제일 마지막줄에 복붙할것

```java
// https://mvnrepository.com/artifact/org.json/json
implementation group: 'org.json', name: 'json', version: '20160810'
```

<br><br><br><br>

# 5. 과제
## 5-1. Person.java와 PersonController.java 만들어서 localhost:8080/myinfo에 내 정보가 뜨도록 만들기.
### 5-1-1. Person.java

```java
public class Person {

    private String name;
    private int age;
    private String address;
    private String job;

    public String getName(){ return this.name; }
    public int getAge(){ return this.age; }
    public String getAddress(){ return this.address; }
    public String getJob(){ return this.job; }

    public void setName(String name){ this.name = name; }
    public void setAge(int age){ this.age = age; }
    public void setAddress(String address){ this.address = address; }
    public void setJob(String job){ this.job = job; }

}
```


### 5-1-2. PersonController.java

```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class PersonController {

    @GetMapping("/myinfo")
    public Person getPerson(){
        Person person = new Person();
        person.setName("나야나");
        person.setAge(200);
        person.setAddress("대한민국");
        person.setJob("개발개발");
        return person;
    }
}
```

### 5-1-3. 결과
![result](/assets/imgss/20210712-SpartaCoding01.JPG)

<br>

#### 만약 이미 8080포트가 사용중일 경우?
- `cmd`창을 연다.
- `netstat -ano` 명령어 입력 
- `0.0.0.0:8080` 의 `PID`를 확인
- `taskkill /f /pid xxxxx` 입력하면 프로세스가 종료된다.

<br><br><br><br>
