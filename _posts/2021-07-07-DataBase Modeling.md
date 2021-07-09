---
title: DataBase Modeling
toc: true
toc_sticky: true
toc_label: "Contents of Page"

categories:
  - Study
tags:
  - SQL_Study
---

<br><br>

# DataBase Modeling
## 1. 모델링 표기법
- IE
- Barker : DA#과 Oracle에서 사용하는 표기법
  * `#` : 식별자
  * `*` : 필수적(Not null)
  * `o` : 선택적(Nullable)
  * `|` : 식별관계 

<br><br>

## 2. 정의순서
- `Entity` - `Relationship` - `Attribute`
- Entity 정의시 핵심적 Attribute는 정의해주어야 한다.

<br><br>

## 3. Entity
### 3-1. 특징
- 집합성 : 2개이상의 식별자를 지녀야함.
- 식별성 : 인스턴스는 식별자에 의해 유일하게 식별가능 해야 한다.
  * 식별자 ( PK, UK ) 
- 사용성 : 업무 프로세스에 필요하고 관리하고자 하는 대상
- 영속성 : 영속적으로 존재하는 데이터 ( 일시적으로 쓰이는 데이터 entity로 정의하지 말것! )
- 관계성 : 다른 엔터티와 최소 1개 이상의 관계
  * **관계표시 생략 엔터티** : 통계성, 코드성, 내부필요한 엔터티

### 3-2. 발생시점에 따른 분류
- Key -> Main -> Action
1. Key
  - 상속받지 않은 홀로 태어난 데이터, 사용자의 입력에 의해 발생.
  - 원래 존재하는 정보, 다른 엔터티와 관계에 의해 생성되지 않고, 독립적으로 생성 가능하고 다른 엔터티의 부모 역할(행위주체/대상)을 한다.
  - Code, 회원, 상품데이터
2. Main
  - 업무의 중심이 되는 데이터
  - 계약, 사고, 주문
3. Action 
  - 2개 이상의 부모로부터 발생. 내용 변경이 많은 데이터 
  - 행위 데이터
  - 주문 목록, 사원 변경 이력(퇴사자 로그)

<br><br>

## 4.Attribute
- 최소단위의 데이터, 의미상 최소단위. 내 조직의 비즈니스룰에 따라 사용할 것인지 판단하에 쪼갠다.
  * 전화번호 = 지역번호 + 번호
  * 주소 = 주소1 + 주소2 
- 복단단다
  * 복합속성
  * 단순속성
  * 단일값(single Value)
  * 다중값(Multiple Value) : 하나의 Attribute에 여러개의 값이 저장될 경우.
    + 1개 : ex) 직장인 대상 마케팅
    + 수평적 N개 : Attribute(Column)를 늘리는 것. Training Null(뒤에있는 공간을 Null로, 3byte차지).
    + 수직적 N개 : Row를 늘리는 것. Entity를 추가.


<br><br><br><br>
