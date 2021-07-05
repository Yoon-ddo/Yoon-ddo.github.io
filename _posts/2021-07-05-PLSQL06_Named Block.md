---
title: PLSQL06_Named Block
toc: true
toc_sticky: true
toc_label: "Contents of Page"

categories:
  - Study
tags:
  - SQL_Study
---

<br><br>

# Named Block
## 1. Block유형
- Anonymous Block
- Named Block(=Stored Block)
  * function
  * procedure
  * package ( = class )
  * trigger 

## 2. function
### 2-1. 함수 정의
- C:/SQLDEV/SQLF/PLSQL/function.sql

 ```sql
 CREATE OR REPLACE FUNCTION CALC_BONUS(P_SALARY IN NUMBER, P_DEPTNO IN NUMBER)
                    -- 파라미터 정의시 LENGTH정의 X, TYPE만 정의한다!
                    -- LENGTH는 코딩으로 제어
RETURN NUMBER -- RETURN필수.
IS
    V_BONUS_RATE        NUMBER :=0;
    V_BONUS             NUMBER(7,2) :=0;
BEGIN
    --근무부서별 Bonus차등 지급
    IF P_DEPTNO = 10 THEN
        V_BONUS_RATE := 0.1;
    ELSIF P_DEPTNO = 20 THEN
        V_BONUS_RATE := 0.2;
    ELSIF P_DEPTNO = 30 THEN
        V_BONUS_RATE := 0.3;
    ELSE
        V_BONUS_RATE := 0.05;
    END IF;
    V_BONUS := ROUND(NVL(P_SALARY,0) * V_BONUS_RATE, 2);
    RETURN V_BONUS;
END CALC_BONUS;
/
```

<br>

### 2-2. 함수 확인

```sql
-- 함수 실행
@C:/SQLDEV/SQLF/PLSQL/function.sql

DESC CALC_BONUS;

--함수명 대소문자 구분하기 때문에 NAME='calc_bonus'는 불가능!
SELECT NAME, LINE, TEXT FROM USER_SOURCE WHERE NAME='CALC_BONUS';
```

<br>

### 2-3. WRAP
- TEXT 소스를 BINARY SOURCE로 변환.
- TEXT SOURCE상태로 배포시 내부 기술 유출될 수 있음.

```sql
WRAP IN function.sql
```

- 자동으로 .plb형태의 파일로 출력됨. 
- 개발 진행중인 상태에서는 TEXT SOURCE가 효율적.
- 배포시 BINARY SOURCE상태로 배포 (WRAP)
- BINARY 상태에서 배포되므로, 원본 소스 관리 주의 필요.
- TEXT SOURCE는 원본 소스 손상시 데이터 딕셔너리에 저장되어 있어 언제든 추출 가능
- 상용화되는 특정 앱에만 한정적으로 사용한다.
- 일반 응용 앱에 사용시 유지보수할 때 문제 초래.


<br><br><br><br>
