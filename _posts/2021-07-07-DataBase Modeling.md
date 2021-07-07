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
- 영속성 : 영속적으로 존재하는 데이터
- 관계성 : 다른 엔터티와 최소 1개 이상의 관계
  * 관계표시 생략 엔터티 : 통계성, 코드성, 내부필요한 엔터티

<br><br><br><br>
