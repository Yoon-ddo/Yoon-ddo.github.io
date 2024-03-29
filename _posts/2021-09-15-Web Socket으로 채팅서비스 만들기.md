---
title: Web Socket으로 채팅서비스 만들기
toc: true
toc_sticky: true
toc_label: "Contents of Page"
categories:
  - Project
tags:
  - Web_Project
---

<br><br>

# Web Socket이란?
* 실시간 채팅 구현 시 http통신의 단점을 보완하기 위해 사용

## 1. Http통신의 절차
1. 고객이 서버에 메시지를 전송
2. 서버는 상담사에게 전송
3. 상담사는 자신에게 온 메시지를 확인

* http통신은 client가 요청을 해야 server가 응답하는 방식이다.
* server가 상담사에게 메시지가 도착했다는 것을 알리려고 해도 request가 없으므로 일방적으로 server가 response 할 수 없다.
* 상담사는 메시지가 왔는지 안왔는지 모르기 때문에 server에 전송된 메시지의 데이터를 request할 수 없음
* 임시방편으로 아래와 같은 방법으로 해결할 수 있으나 문제점이 있다.
  - ajax를 이용 : 주기적으로 한 페이지 안에서 server에 자신에게 보낼 정보가 있는지 request
    + 문제 : ajax를 주기적으로 해야해서 무리가 간다. 실시간도 아님
  - 페이지 이동할때마다 자신에게 온 정보 확인하도록 request
    + 문제 : 페이지 이동을 하기 때문에 채팅이라고 할 수 없음


## 2. Web Socket
* Web Socket은 IETF에 의해 RFC 6455로 표준화된 기술이다.
* http와 다르게 이중통신을 지원
* client의 요청 없이도 server에서 먼저 client에게 정보를 전송
* 영구적 양방향 통신이다.
* HTML 5의 주요 API


<br><br>

# 코드 구현하기
## 1. 의존성 추가
* pom.xml 파일에 라이브러리를 추가한다.
  - jackson-databind : 웹소켓에서 json형식으로 주고받기 때문에 추가해야함
  - spring-websocket : Spring에서 웹소켓기능을 사용할 수 있게함.
  - slf4j-api

```xml
<!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-databind -->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.12.3</version>
</dependency>

<!-- https://mvnrepository.com/artifact/org.springframework/spring-websocket -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-websocket</artifactId>
    <version>5.3.9</version>
</dependency>

<!-- https://mvnrepository.com/artifact/javax.websocket/javax.websocket-api -->
<dependency>
    <groupId>javax.websocket</groupId>
    <artifactId>javax.websocket-api</artifactId>
    <version>1.1</version>
    <scope>provided</scope>
</dependency>


<!-- 로깅 관련 -->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.25</version>
</dependency>
```

### 2-1 web.xml  웹소켓 설정(비동기 지원)
* `<servlet>`태그안 제일 마지막에 아래의 태그를 추가한다.

```xml
<async-supported>true</async-supported>
```

<br><br>

## 2. bean등록
* spring-mvc.xml
  - `<beans>`를 수정하고
  - `<websocket:handlers>`와 `<bean>`객체를 추가한다.

```xml
<!-- beans에 추가 -->
xmlns:websocket="http://www.springframework.org/schema/websocket"

<!-- beans태그 안에 추가 -->
<!-- Handler가 작동하는 객체등록 -->
<bean id="consultHandler" class="kr.co.hana.consult.handler.ConsultHandler" />

<!-- Websocket과 handler 매핑
  path : 요청 uri 
  id : Handler의 bean객체 id와 일치해야함
-->
<websocket:handlers >
    <websocket:mapping handler="consultHandler" path="/hanaconsultPrac" />
    <websocket:sockjs websocket-enabled="true" />
</websocket:handlers>
```

<br><br>

## 3. Handler 클래스와 Config 생성
* ConsultHandler.java

```java
package kr.co.hana.consult.handler;

import java.util.ArrayList;
import java.util.List;

import org.springframework.web.socket.CloseStatus;
import org.springframework.web.socket.TextMessage;
import org.springframework.web.socket.WebSocketSession;
import org.springframework.web.socket.handler.TextWebSocketHandler;


public class ConsultHandler extends TextWebSocketHandler {

private List<WebSocketSession> sessionList = new ArrayList<WebSocketSession>();
	
	@Override
	public void afterConnectionEstablished(WebSocketSession session) throws Exception {
		/*
        채팅을 위해 페이지에 연결 후 해당 클라이언트의 세션을 sessionList에 add
     */
		sessionList.add(session);
	}
	
	@Override
	protected void handleTextMessage(WebSocketSession session, TextMessage message) throws Exception {
		for(WebSocketSession sess: sessionList) {
      /*
        웹소켓 서버로 메시지를 전송했을 때 이 메서드가 호출됨. 
        
      */
			//System.out.println("handleTextMessage" + message);
			sess.sendMessage(new TextMessage(session.getId()+": "+message.getPayload()));
		}
	}
	
	@Override
	public void afterConnectionClosed(WebSocketSession session, CloseStatus status) throws Exception {
    /*
      연결이 끊어진 경우 (채팅방 나간 경우) remove로 해당 세션 제거
    */
		sessionList.remove(session);
	}
}
```

<br><br>

## 4. Controller 등록
* 나는 특정 uri 요청시 실행하게 하고 싶었기 때문에 Controller를 설정했다.

```java
package kr.co.hana.consult.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class ConsultController {
	

	@GetMapping("/hanaconsultPrac")
	public String OnlineConsult2() throws Exception {
		
		System.out.println("상담받아봥");
		return "consult/consultprac";
	}

}
```

<br><br>

## 5. 화면구성

~~~html

<%@ page language="java" contentType="text/html; charset=UTF-8"
	pageEncoding="UTF-8"%>

<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
<html>
<head>
<title>Home</title>
<meta charset="UTF-8" />
<script
	src="https://ajax.googleapis.com/ajax/libs/jquery/3.3.1/jquery.min.js"></script>
<script src="${ pageContext.request.contextPath }/resources/js/sockjs.min.js"></script>
</head>
<body>
	<form id="chatForm">
		<input type="text" id="message" />
		<button>send</button>
	</form>
	<div id="chat"></div>
	<script>
		$(document).ready(function() {
			$("#chatForm").submit(function(event) {
				event.preventDefault();
				sock.send($("#message").val());
				$("#message").val('').focus();
			});
		});
    
    // 로컬에서만 작동하게 하려면 localhost:포트번호도 가능.
    // 다른컴퓨터에서 작동하게하려면 cmd -> ipconfig에서 내아이피확인 후 입력.
		var sock = new SockJS("http://내아이피:포트번호/프로젝트이름/uri");
		sock.onmessage = function(e) {
			$("#chat").append(e.data + "<br/>");
		}

		sock.onclose = function() {
			$("#chat").append("연결 종료");
		}
	</script>
</body>
</html>

~~~

<br><br>

# Stomp
* [참고블로그](https://inspirit941.tistory.com/343)
* [참고Github](https://github.com/Java-Techie-jt/Spring-Boot-WebSocket)
* WebSocket은 단지 통신 프로토콜이다.
* Stomp : Streaming Text Oriented Messaging Protocol
* 프로토콜을 지원하는 Message Broker와 Stomp Client간 통신을 지ㅜ언
* Spring은 default로 지원한다.
* 다른 Messaging Protocol (RabbitMQ / ActiveMQ)등도 사용 가능

## 구현순서
1. WebSocket Config 클래스를 생성 (WebSocketMessageBrokerConfigurer 인터페이스 상속)
2. 메시지 클래스 생성 ( VO )
3. Controller 생성 ( 클라이언트로부터 메시지를 받아 처리 )
4. 프론트 구성

<br><br>



# Issues
## 1. 내 컴퓨터에서는 정상작동되는데 다른사람컴퓨터에서 내 프로젝트 접근안되는 문제가 발생
* 방화벽 해제했더니 된다 ㅋ...
* 방화벽 해제 방법
  1. `시작` - `방화벽 상태 확인`
  2. 왼쪽메뉴 중 `Windows Defender 방화벽 설정 또는 해제`
  3. Windows Defender 방화벽 사용안함 (권장하지 않음) 두개 모두 체크 후 확인

* 그런데 <span style="color:red;">방화벽 해제하면 위험한거아닌가?</span> 에 대한 궁금증이 생겼다.
- 교수님께서 방화벽 해제가 아니라 포트를 열라고 하셧다... 이게아닌가? ㅠㅋㅋㅋ

<br>

## 2. 메시지 중복에러
* 위의 코드를 응용하여 Template에 적용시켰더니 메시지가 두번 찍히는 에러가 발생했다.
* 받는 사람 화면에서는 잘 출력되는데 내가 보낸 메시지가 내쪽, 상대쪽에 두번 찍히는 에러발생
* 어떻게 해결할까...ㅠ












<br><br><br><br>
