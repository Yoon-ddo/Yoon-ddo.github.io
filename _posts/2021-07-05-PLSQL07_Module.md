---
title: PLSQL07_Module(Stored Block2)
toc: true
toc_sticky: true
toc_label: "Contents of Page"

categories:
  - Study
tags:
  - SQL_Study
---

<br><br>

# Module (Stored Block2)
## 1. 변수 선언
- :new_bal과 new_bal의 차이
  * host language의 변수를 사용하기 위해서는 콜론(:)을 붙여야 사용할 수 있다.
  * (:) 가 없는 new_bal은 PL/SQL에서 정의한 변수이다. 

<br><br><br><br>

## 2. VARIABLE &PRINT
### 2-1. VARIABLE
- PL/SQL 블록에서 참조할 수 있는 BIND 변수 선언
- 선언된 BIND 변수 조회
### 2-2. PRINT
- BIND 변수의 값을 SQL\*PLUS화면에 출력하는 기능

```sql
-- 변수 선언 : bind변수 = host변수 = 글로벌변수
VARIABLE H_SALARY   NUMBER
VARIABLE H_TAX      NUMBER
DECLARE
    C_TAX_RATE NUMBER(2,3);
BEGIN
    C_TAX_RATE := 0.05; -- 근로소득세율 (Local)
    :H_SALARY := 1000;  -- 급여 (Host)
    
    :H_TAX := ROUND(:H_SALARY * C_TAX_RATE,2);
END;
/

-- 변수 출력 : bind변수 = host변수 = 글로벌변수
PRINT H_SALARY -- 1000
PRINT H_TAX   -- 50
```



<br><br><br><br>


