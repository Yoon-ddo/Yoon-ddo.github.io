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


# 1. 네이버 API 이용하기
## 1-1. 쇼핑 API 신청
* https://developers.naver.com/docs/search/shopping/
* 로그인- 약관동의 - 계정정보등록 - 애플리케이션등록 모두 하기
* 서비스환경추가 -> `WEB설정` : http://localhost
* 생성완료되면 Client ID와 Secret 가 나타남

## 1-2. 바로 사용하기
* https://developers.naver.com/docs/search/shopping/
* 설명문서 참고


## 1-3. API Java로 실행

```java
package com.example.week04.utils;

import org.springframework.http.*;
import org.springframework.web.client.RestTemplate;

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
