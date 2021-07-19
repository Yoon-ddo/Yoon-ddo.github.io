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

# 3. SQL연습
* courses테이블 생성

```sql
CREATE TABLE IF NOT EXISTS courses (
    id bigint(5) NOT NULL AUTO_INCREMENT, 
    title varchar(255) NOT NULL,
    tutor varchar(255) NOT NULL,
    PRIMARY KEY (id)
);
```

* 데이터 삽입

```sql
INSERT INTO courses (title, tutor) VALUES
    ('웹개발의 봄, Spring', '남병관'), ('웹개발 종합반', '이범규');
```

* 데이터 조회

```sql
SELECT * FROM courses;
```


<br><br><br>


# 4. JPA
- JPA는 SQL을 쓰지 않고 데이터를 생성, 조회, 수정, 삭제할 수 있도록 해주는 번역기
- `JPA가 없다면?` String query = "SELECT * FROM EMP WHERE ID = ? ";
- `JPA가 있다면?` implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
                  repository.save(new Customer("Jack", "Bauer"));

# 4-1. Domain, Repository
* Table은  Domain, SQL은 Repository
* src > main > java에 domain패키지 생성
  -  Course (Java클래스 생성)
  -  CourseRepository (Interface생성)

* Course.java (에러시 Import확인 alt + enter)

```java
package com.sparta.week02.domain;

import lombok.NoArgsConstructor;

import javax.persistence.*;

public class Course {

    @NoArgsConstructor // 기본생성자를 대신 생성해줍니다.
    @Entity // 테이블임을 나타냅니다.
    public class Course {

        @Id // ID 값, Primary Key로 사용하겠다는 뜻입니다.
        @GeneratedValue(strategy = GenerationType.AUTO) // 자동 증가 명령입니다.
        private Long id;

        @Column(nullable = false) // 컬럼 값이고 반드시 값이 존재해야 함을 나타냅니다.
        private String title;

        @Column(nullable = false)
        private String tutor;

        public String getTitle() {
            return this.title;
        }

        public String getTutor() {
            return this.tutor;
        }

        public Course(String title, String tutor) {
            this.title = title;
            this.tutor = tutor;
        }
    }
}
```

* CourseRepository.java

```java
package com.sparta.week02.domain;

import org.springframework.data.jpa.repository.JpaRepository;

public interface CourseRepository extends JpaRepository<Course, Long> {
    // jpaRepository에 있는걸 가져와서 Course에서 쓸것임. <클래스, 자료형>
}
```

<br><br><br>

# 5. JPA사용
## 5-1. src > main > resources > application.properties 세팅
- spring.jpa.show-sql=true 붙여넣기

## 5-2. Week02.Application세팅

```java
    // Week02Application.java 의 main 함수 아래에 붙여주세요.
    @Bean
    public CommandLineRunner demo(CourseRepository repository) {
        return (args) -> {

            Course course1 = new Course("웹개발의 봄 Spring", "남병관");
            repository.save(course1);
            List<Course> list = repository.findAll();
            for (int i=0; i<list.size(); i++) {
                System.out.println(list.get(i));
            }

        };
    }
```

<br><br><br>

# 6. 생성일자, 수정일자
- Timestamped.java

```java
package com.sparta.week02.domain;

import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.LastModifiedDate;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;

import javax.persistence.EntityListeners;
import javax.persistence.MappedSuperclass;
import java.time.LocalDateTime;

@MappedSuperclass // 상속했을 때, 컬럼으로 인식하게 합니다.
@EntityListeners(AuditingEntityListener.class) // 생성/수정 시간을 자동으로 반영하도록 설정
public abstract class Timestamped {

    @CreatedDate // 생성일자임을 나타냅니다.
    private LocalDateTime createdAt;

    @LastModifiedDate // 마지막 수정일자임을 나타냅니다.
    private LocalDateTime modifiedAt;
}
```

- Application에 `@EnableJpaAuditing` 추가.









