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

<br>

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

<br>

* Product.java

```java
package com.example.week04.models;

import lombok.Getter;
import lombok.NoArgsConstructor;

import javax.persistence.*;

@Getter // get 함수를 일괄적으로 만들어줍니다.
@NoArgsConstructor // 기본 생성자를 만들어줍니다.
@Entity // DB 테이블 역할을 합니다.
public class Product extends Timestamp{

    // ID가 자동으로 생성 및 증가합니다.
    @GeneratedValue(strategy = GenerationType.AUTO)
    @Id
    private Long id;

    // 반드시 값을 가지도록 합니다.
    @Column(nullable = false)
    private String title;

    @Column(nullable = false)
    private String image;

    @Column(nullable = false)
    private String link;

    @Column(nullable = false)
    private int lprice;

    @Column(nullable = false)
    private int myprice;
}
```

<br>

* ProductRepository (인터페이스)

```java
package com.example.week04.models;

import org.springframework.data.jpa.repository.JpaRepository;

public interface ProductRepository extends JpaRepository<Product, Long> {


}
```

<br>

* ProductRestController.java

```java
package com.example.week04.controller;

import com.example.week04.models.Product;
import com.example.week04.models.ProductRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RequiredArgsConstructor // final로 선언된 멤버 변수를 자동으로 생성합니다.
@RestController // JSON으로 데이터를 주고받음을 선언합니다.
public class ProductRestController {

    private final ProductRepository productRepository;

    // 등록된 전체 상품 목록 조회
    @GetMapping("/api/products")
    public List<Product> getProducts() {
        return productRepository.findAll();
    }
}
```

<br><br>

## 2. 관심상품 등록
* ProductRequestDto.java

```java
package com.example.week04.models;

import lombok.Getter;

@Getter
public class ProductRequestDto {
    private String title;
    private String link;
    private String image;
    private int lprice;
}
```

<br>

* ProductMypriceRequestDto.java

```java
package com.example.week04.models;

import lombok.Getter;

@Getter
public class ProductMypriceRequestDto {
    private int myprice;
}
```

<br>

* Product.java

```java
package com.example.week04.models;

import lombok.Getter;
import lombok.NoArgsConstructor;

import javax.persistence.*;

@Getter // get 함수를 일괄적으로 만들어줍니다.
@NoArgsConstructor // 기본 생성자를 만들어줍니다.
@Entity // DB 테이블 역할을 합니다.
public class Product extends Timestamp{

    // ID가 자동으로 생성 및 증가합니다.
    @GeneratedValue(strategy = GenerationType.AUTO)
    @Id
    private Long id;

    // 반드시 값을 가지도록 합니다.
    @Column(nullable = false)
    private String title;

    @Column(nullable = false)
    private String image;

    @Column(nullable = false)
    private String link;

    @Column(nullable = false)
    private int lprice;

    @Column(nullable = false)
    private int myprice;

    // 관심 상품 생성 시 이용합니다.
    public Product(ProductRequestDto requestDto) {
        this.title = requestDto.getTitle();
        this.image = requestDto.getImage();
        this.link = requestDto.getLink();
        this.lprice = requestDto.getLprice();
        this.myprice = 0;
    }

    // 관심 가격 변경 시 이용합니다.
    public void update(ProductMypriceRequestDto requestDto) {
        this.myprice = requestDto.getMyprice();
    }
}
```

<br>

* ProductService.java

```java 
package com.example.week04.service;

import com.example.week04.models.Product;
import com.example.week04.models.ProductMypriceRequestDto;
import com.example.week04.models.ProductRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;

import javax.transaction.Transactional;

@RequiredArgsConstructor // final로 선언된 멤버 변수를 자동으로 생성합니다.
@Service // 서비스임을 선언합니다.
public class ProductService {

    private final ProductRepository productRepository;

    @Transactional // 메소드 동작이 SQL 쿼리문임을 선언합니다.
    public Long update(Long id, ProductMypriceRequestDto requestDto) {
        Product product = productRepository.findById(id).orElseThrow(
                () -> new NullPointerException("해당 아이디가 존재하지 않습니다.")
        );
        product.update(requestDto);
        return id;
    }
}
```

<br>

* ProductRestController.java

```java
package com.example.week04.controller;

import com.example.week04.models.Product;
import com.example.week04.models.ProductRepository;
import com.example.week04.models.ProductRequestDto;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RequiredArgsConstructor // final로 선언된 멤버 변수를 자동으로 생성합니다.
@RestController // JSON으로 데이터를 주고받음을 선언합니다.
public class ProductRestController {

    private final ProductRepository productRepository;

    // 등록된 전체 상품 목록 조회
    @GetMapping("/api/products")
    public List<Product> getProducts() {
        return productRepository.findAll();
    }

    @PostMapping("/api/products")
    public Product createProduct(@RequestBody ProductRequestDto requestDto){
        Product product = new Product(requestDto);
        return productRepository.save(product);
    }

}
```

<br><br>

## 3. 키워드로 상품 검색
* Gradle에 추가

```java
// https://mvnrepository.com/artifact/org.json/json
    implementation group: 'org.json', name: 'json', version: '20160810'
```

<br>

* ItemDto.java 생성

```java
package com.example.week04.models;

import lombok.Getter;
import org.json.JSONObject;

@Getter
public class ItemDto {
    private String title;
    private String link;
    private String image;
    private int lprice;

    public ItemDto(JSONObject itemJson) {
        this.title = itemJson.getString("title");
        this.link = itemJson.getString("link");
        this.image = itemJson.getString("image");
        this.lprice = itemJson.getInt("lprice");
    }
}
```

<br>

* NaverShopSearch.java (수정)

```java
package com.example.week04.utils;

import com.example.week04.models.ItemDto;
import org.json.JSONArray;
import org.json.JSONObject;
import org.springframework.http.*;
import org.springframework.stereotype.Component;
import org.springframework.web.client.RestTemplate;

import java.util.ArrayList;
import java.util.List;

@Component // 스프링이 자동으로 필요한 클래스를 필요한 곳에 생성하도록 권한 부여
public class NaverShopSearch {
    public String search(String query) {
        RestTemplate rest = new RestTemplate();
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Naver-Client-Id", "나의키");
        headers.add("X-Naver-Client-Secret", "rodgIc6tma");
        String body = "";

        HttpEntity<String> requestEntity = new HttpEntity<String>(body, headers);
        ResponseEntity<String> responseEntity = rest.exchange("https://openapi.naver.com/v1/search/shop.json?query=" + query, HttpMethod.GET, requestEntity, String.class);
        HttpStatus httpStatus = responseEntity.getStatusCode();
        int status = httpStatus.value();
        String response = responseEntity.getBody();
        System.out.println("Response status: " + status);
        System.out.println(response);

        return response;
    }




    public List<ItemDto> fromJSONtoItems(String result) {
        JSONObject rjson = new JSONObject(result);
        JSONArray items  = rjson.getJSONArray("items");
        List<ItemDto> ret = new ArrayList<>();
        for (int i=0; i<items.length(); i++) {
            JSONObject itemJson = items.getJSONObject(i);
            System.out.println(itemJson);
            ItemDto itemDto = new ItemDto(itemJson);
            ret.add(itemDto);
        }
        return ret;
    }

    public static void main(String[] args) {
        NaverShopSearch naverShopSearch = new NaverShopSearch();
        String ret = naverShopSearch.search("아이맥");
        naverShopSearch.fromJSONtoItems(ret);
    }
}


```

<br>

* SearchRequestController.java

```java
package com.example.week04.controller;

import com.example.week04.models.ItemDto;
import com.example.week04.utils.NaverShopSearch;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RequiredArgsConstructor // final 로 선언된 클래스를 자동으로 생성합니다.
@RestController // JSON으로 응답함을 선언합니다.
public class SearchRequestController {

    private final NaverShopSearch naverShopSearch;

    @GetMapping("/api/search")
    public List<ItemDto> getItems(@RequestParam String query) {
        String resultString = naverShopSearch.search(query);
        return naverShopSearch.fromJSONtoItems(resultString);
    }
}
```

<br><br>

## 4. 프론트 연결
* 파일 분리
  1. css, js파일 만들기
  2. link와 script태그로 파일불러오기 


* index.html

~~~html

<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <link rel="stylesheet" href="style.css">
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.5.1/jquery.min.js"></script>
    <script src="basic.js"></script>
    <title>나만의 셀렉샵</title>
</head>
<body>
<div class="header">
    Select Shop
</div>
<div class="nav">
    <div class="nav-see active">
        모아보기
    </div>
    <div class="nav-search">
        탐색하기
    </div>
</div>
<div id="see-area">
    <div id="product-container">
        <div class="product-card" onclick="window.location.href='https://spartacodingclub.kr'">
            <div class="card-header">
                <img src="https://shopping-phinf.pstatic.net/main_2085830/20858302247.20200602150427.jpg?type=f300"
                     alt="">
            </div>
            <div class="card-body">
                <div class="title">
                    Apple 아이폰 11 128GB [자급제]
                </div>
                <div class="lprice">
                    <span>919,990</span>원
                </div>
                <div class="isgood">
                    최저가
                </div>
            </div>
        </div>
    </div>
</div>
<div id="search-area">
    <div>
        <input type="text" id="query">
        <!--    <img src="images/icon-search.png" alt="">-->
    </div>
    <div id="search-result-box">
        <div class="search-itemDto">
            <div class="search-itemDto-left">
                <img src="https://shopping-phinf.pstatic.net/main_2399616/23996167522.20200922132620.jpg?type=f300" alt="">
            </div>
            <div class="search-itemDto-center">
                <div>Apple 아이맥 27형 2020년형 (MXWT2KH/A)</div>
                <div class="price">
                    2,289,780
                    <span class="unit">원</span>
                </div>
            </div>
            <div class="search-itemDto-right">
                <img src="images/icon-save.png" alt="" onclick='addProduct()'>
            </div>
        </div>
    </div>
    <div id="container" class="popup-container">
        <div class="popup">
            <button id="close" class="close">
                X
            </button>
            <h1>⏰최저가 설정하기</h1>
            <p>최저가를 설정해두면 선택하신 상품의 최저가가 떴을 때<br/> 표시해드려요!</p>
            <div>
                <input type="text" id="myprice" placeholder="200,000">원
            </div>
            <button class="cta" onclick="setMyprice()">설정하기</button>
        </div>
    </div>
</div>
</body>
</html>

~~~

<br>

* basic.js

```javascript
let targetId;

$(document).ready(function () {
    // id 가 query 인 녀석 위에서 엔터를 누르면 execSearch() 함수를 실행하라는 뜻입니다.
    $('#query').on('keypress', function (e) {
        if (e.key == 'Enter') {
            execSearch();
        }
    });
    $('#close').on('click', function () {
        $('#container').removeClass('active');
    })

    $('.nav div.nav-see').on('click', function () {
        $('div.nav-see').addClass('active');
        $('div.nav-search').removeClass('active');

        $('#see-area').show();
        $('#search-area').hide();
    })
    $('.nav div.nav-search').on('click', function () {
        $('div.nav-see').removeClass('active');
        $('div.nav-search').addClass('active');

        $('#see-area').hide();
        $('#search-area').show();
    })

    $('#see-area').show();
    $('#search-area').hide();

    showProduct();
})


function numberWithCommas(x) {
    return x.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",");
}

//////////////////////////////////////////////////////////////////////////////////////////
///// 여기 아래에서부터 코드를 작성합니다. ////////////////////////////////////////////////////////
//////////////////////////////////////////////////////////////////////////////////////////

function execSearch() {
    /**
     * 검색어 input id: query
     * 검색결과 목록: #search-result-box
     * 검색결과 HTML 만드는 함수: addHTML
     */
    // 1. 검색창의 입력값을 가져온다.
    // 2. 검색창 입력값을 검사하고, 입력하지 않았을 경우 focus.
    // 3. GET /api/search?query=${query} 요청
    // 4. for 문마다 itemDto를 꺼내서 HTML 만들고 검색결과 목록에 붙이기!

}

function addHTML(itemDto) {
    /**
     * class="search-itemDto" 인 녀석에서
     * image, title, lprice, addProduct 활용하기
     * 참고) onclick='addProduct(${JSON.stringify(itemDto)})'
     */
    return ``
}

function addProduct(itemDto) {
    /**
     * modal 뜨게 하는 법: $('#container').addClass('active');
     * data를 ajax로 전달할 때는 두 가지가 매우 중요
     * 1. contentType: "application/json",
     * 2. data: JSON.stringify(itemDto),
     */
    // 1. POST /api/products 에 관심 상품 생성 요청
    // 2. 응답 함수에서 modal을 뜨게 하고, targetId 를 reponse.id 로 설정 (숙제로 myprice 설정하기 위함)
}

function showProduct() {
    /**
     * 관심상품 목록: #product-container
     * 검색결과 목록: #search-result-box
     * 관심상품 HTML 만드는 함수: addProductItem
     */
    // 1. GET /api/products 요청
    // 2. 관심상품 목록, 검색결과 목록 비우기
    // 3. for 문마다 관심 상품 HTML 만들어서 관심상품 목록에 붙이기!
}

function addProductItem(product) {
    // link, image, title, lprice, myprice 변수 활용하기
    return ``;
}

function setMyprice() {
    /**
     * 숙제! myprice 값 설정하기.
     * 1. id가 myprice 인 input 태그에서 값을 가져온다.
     * 2. 만약 값을 입력하지 않았으면 alert를 띄우고 중단한다.
     * 3. PUT /api/product/${targetId} 에 data를 전달한다.
     *    주의) contentType: "application/json",
     *         data: JSON.stringify({myprice: myprice}),
     *         빠뜨리지 말 것!
     * 4. 모달을 종료한다. $('#container').removeClass('active');
     * 5, 성공적으로 등록되었음을 알리는 alert를 띄운다.
     * 6. 창을 새로고침한다. window.location.reload();
     */
}
```

<br>

* style.css

```css

body {
    margin: 0px;
}

#search-result-box {
    margin-top: 15px;
}

.search-itemDto {
    width: 530px;
    display: flex;
    flex-direction: row;
    align-content: center;
    justify-content: space-around;
}

.search-itemDto-left img {
    width: 159px;
    height: 159px;
}

.search-itemDto-center {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: space-evenly;
}

.search-itemDto-center div {
    width: 280px;
    height: 23px;
    font-size: 18px;
    font-weight: normal;
    font-stretch: normal;
    font-style: normal;
    line-height: 1.3;
    letter-spacing: -0.9px;
    text-align: left;
    color: #343a40;
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
}

.search-itemDto-center div.price {
    height: 27px;
    font-size: 27px;
    font-weight: 600;
    font-stretch: normal;
    font-style: normal;
    line-height: 1;
    letter-spacing: -0.54px;
    text-align: left;
    color: #E8344E;
}

.search-itemDto-center span.unit {
    width: 17px;
    height: 18px;
    font-size: 18px;
    font-weight: 500;
    font-stretch: normal;
    font-style: normal;
    line-height: 1;
    letter-spacing: -0.9px;
    text-align: center;
    color: #000000;
}

.search-itemDto-right {
    display: inline-block;
    height: 100%;
    vertical-align: middle
}

.search-itemDto-right img {
    height: 25px;
    width: 25px;
    vertical-align: middle;
    margin-top: 60px;
    cursor: pointer;
}

input#query {
    padding: 15px;
    width: 526px;
    border-radius: 2px;
    background-color: #e9ecef;
    border: none;

    background-image: url('images/icon-search.png');
    background-repeat: no-repeat;
    background-position: right 10px center;
    background-size: 20px 20px;
}

input#query::placeholder {
    padding: 15px;
}

button {
    color: white;
    border-radius: 4px;
    border-radius: none;
}

.popup-container {
    display: none;
    position: fixed;
    top: 0;
    left: 0;
    bottom: 0;
    right: 0;
    background-color: rgba(0, 0, 0, 0.5);
    align-items: center;
    justify-content: center;
}

.popup-container.active {
    display: flex;
}

.popup {
    padding: 20px;
    box-shadow: 2px 2px 10px rgba(0, 0, 0, 0.3);
    position: relative;
    width: 370px;
    height: 190px;
    border-radius: 11px;
    background-color: #ffffff;
}

.popup h1 {
    margin: 0px;
    font-size: 22px;
    font-weight: 500;
    font-stretch: normal;
    font-style: normal;
    line-height: 1;
    letter-spacing: -1.1px;
    color: #000000;
}

.popup input {
    width: 330px;
    height: 39px;
    border-radius: 2px;
    border: solid 1.1px #dee2e6;
    margin-right: 9px;
    margin-bottom: 10px;
    padding-left: 10px;
}

.popup button.close {
    position: absolute;
    top: 15px;
    right: 15px;
    color: #adb5bd;
    background-color: #fff;
    font-size: 19px;
    border: none;
}

.popup button.cta {
    width: 369.1px;
    height: 43.9px;
    border-radius: 2px;
    background-color: #15aabf;
    border: none;
}

#search-area, #see-area {
    width: 530px;
    margin: auto;
}

.nav {
    width: 530px;
    margin: 30px auto;
    display: flex;
    align-items: center;
    justify-content: space-around;
}

.nav div {
    cursor: pointer;
}

.nav div.active {
    font-weight: 700;
}

.header {
    background-color: #15aabf;
    color: white;
    text-align: center;
    padding: 50px;
    font-size: 45px;
    font-weight: bold;
}

#product-container {
    grid-template-columns: 100px 50px 100px;
    grid-template-rows: 80px auto 80px;
    column-gap: 10px;
    row-gap: 15px;
}

.product-card {
    width: 300px;
    margin: auto;
    cursor: pointer;
}

.product-card .card-header {
    width: 300px;
}

.product-card .card-header img {
    width: 300px;
}

.product-card .card-body {
    margin-top: 15px;
}

.product-card .card-body .title {
    font-size: 15px;
    font-weight: normal;
    font-stretch: normal;
    font-style: normal;
    line-height: 1;
    letter-spacing: -0.75px;
    text-align: left;
    color: #343a40;
    margin-bottom: 10px;
}

.product-card .card-body .lprice {
    font-size: 15.8px;
    font-weight: normal;
    font-stretch: normal;
    font-style: normal;
    line-height: 1;
    letter-spacing: -0.79px;
    color: #000000;
    margin-bottom: 10px;
}

.product-card .card-body .lprice span {
    font-size: 21.4px;
    font-weight: 600;
    font-stretch: normal;
    font-style: normal;
    line-height: 1;
    letter-spacing: -0.43px;
    text-align: left;
    color: #E8344E;
}

.product-card .card-body .isgood {
    margin-top: 10px;
    padding: 10px 20px;
    color: white;
    border-radius: 2.6px;
    background-color: #ff8787;
    width: 42px;
}

.none {
    display: none;
}

```

