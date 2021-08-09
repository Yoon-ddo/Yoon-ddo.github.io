---
title: SpartaCoding_웹개발의 봄, Spring 4주차
toc: true
toc_sticky: true
toc_label: "Contents of Page"

categories:
  - Study
tags:
  - SpartaCoding_Study
---

<br><br>

# 네이버 쇼핑 API
* 인증실패로 일주일 내내 고생하다가 결국 애플리케이션 다시만들었더니 해결되었다...ㅋ


```java
public class NaverShopSearch {
    public String search() {
        RestTemplate rest = new RestTemplate();
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Naver-Client-Id", "여러분이 발급받은 Client ID");
        headers.add("X-Naver-Client-Secret", "여러분이 발급받은 Client Secret");
        String body = "";

        HttpEntity<String> requestEntity = new HttpEntity<String>(body, headers);
        ResponseEntity<String> responseEntity = rest.exchange("https://openapi.naver.com/v1/search/shop.json?query=adidas", HttpMethod.GET, requestEntity, String.class);
        HttpStatus httpStatus = responseEntity.getStatusCode();
        int status = httpStatus.value();
        String response = responseEntity.getBody();
        System.out.println("Response status: " + status);
        System.out.println(response);

        return response;
    }

    public static void main(String[] args) {
        NaverShopSearch naverShopSearch = new NaverShopSearch();
        naverShopSearch.search();
    }
}
```


# 본격 사용하기
## 1. 관심상품 조회
* Timestamp.java

```java
package com.example.week04.models;

import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.LastModifiedDate;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;

import javax.persistence.EntityListeners;
import javax.persistence.MappedSuperclass;
import java.time.LocalDateTime;

@Getter // get 함수를 자동 생성합니다.
@MappedSuperclass // 멤버 변수가 컬럼이 되도록 합니다.
@EntityListeners(AuditingEntityListener.class) // 변경되었을 때 자동으로 기록합니다.
public abstract class Timestamped {
    @CreatedDate // 최초 생성 시점
    private LocalDateTime createdAt;

    @LastModifiedDate // 마지막 변경 시점
    private LocalDateTime modifiedAt;
}
```

* Week04Application

```java
package com.example.week04;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.data.jpa.repository.config.EnableJpaAuditing;

@EnableJpaAuditing // 시간 자동 변경이 가능하도록 합니다.
@SpringBootApplication // 스프링 부트임을 선언합니다.
public class Week04Application {
    public static void main(String[] args) {
        SpringApplication.run(Week04Application.class, args);
    }
}
```
