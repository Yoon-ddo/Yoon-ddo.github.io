---
title: Project일기03_Web Socket으로 채팅서비스 만들기
toc: true
toc_sticky: true
toc_label: "Contents of Page"
categories:
  - Project
tags:
  - Java_Project
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

<br><br>

# 코드 구현하기
## 1. 의존성 추가
* pom.xml 파일에 라이브러리를 추가한다.

```xml
```











<br><br><br><br>
