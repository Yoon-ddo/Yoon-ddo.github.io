---
title: PLSQL04_Data Type
toc: true
toc_sticky: true
toc_label: "Contents of Page"

categories:
  - Study
tags:
  - SQL_Study
---

<br><br>

# Data Type
## 1. Data Type 개요
* Scalar Data Type
* Composite Data Type
* Reference Data Type
* LOB Data Type
* Object Data Type

<br><br><br>

## 2. Scalar Data Type : 하나의 변수가 하나의 값을 저장.
- VARCHAR2(n)
  * 가변길이 문자 데이터 타입
  * Table의 Column : 1 ~ 4000 Bytes
  * PL/SQL의 변수 : 1 ~ 32767 Bytes   

- NUMBER(p,s)
  * 가변길이 숫자 데이터 타입
  * Table의 Column과 PL/SQL변수가 동일한 특성을 가진다.
  * 38자리 이하의 유효 숫자 자리
  * Packed Decimal (4bit가 1개의 숫자 저장.)  

- PLS_INTEGER
  * Table의 Column : 지원하지 않음
  * PL/SQL의 변수  :  -2147483647 ~ 2147483647 사이의 signed정수에 대한 기본형
  * BINARY_INTEGER Type보다 **빠른 연산 성능개선을 위한 유형**이며 BINARY_INTEGER를 대체하는 Data Type유형

<br><br><br>

## 3. Composite Data Type : 변수 하나가 여러개의 데이터를 저장.
## 4. 참조연산
## 5. Block내의 Select
