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

<br><br>

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

## 3. PROCEDURE
### 3-1. PROCEDURE 생성

```sql
-- 프로시저 생성
CREATE OR REPLACE PROCEDURE CHANGE_SALARY(A_EMPNO IN NUMBER, A_SALARY NUMBER DEFAULT 2000)
AS
BEGIN
    UPDATE EMP
    SET SAL = A_SALARY
    WHERE EMPNO = A_EMPNO;
    COMMIT;
END CHANGE_SALARY;
/

DESC CHANGE_SALARY;

-- 프로시저 테스트 1
VARIABLE P_EMPNO NUMBER
VARIABLE P_SALARY NUMBER
BEGIN
    :P_EMPNO := 7369;
    :P_SALARY := 7369;
    CHANGE_SALARY(:P_EMPNO, :P_SALARY);
END;
/

-- 프로시저 테스트 2
EXECUTE CHANGE_SALARY(:P_EMPNO); -- stored block실행

SELECT EMPNO, SAL FROM EMP WHERE EMPNO = 7369;
```

- EXECUTE
  * BEGIN~ END;구문으로 바꾸어서 ORACLE DBMS에 전송. 

#### 3-1-1. PROCEDURE 생성중 만난 오류

- ORA-04098: 'SCOTT.TRG_EMP_UPD_SAL' 트리거가 부적합하며 재검증을 실패했습니다
- ORA-06512: "SCOTT.CHANGE_SALARY",  4행
- 위와 같은 에러를 만났다.

```sql
-- 내가 생성한 모든 소스 보기
SELECT * FROM USER_SOURCE WHERE NAME='TRG_EMP_UPD_SAL';

-- 내가 만든 소스를 언제만들었는지 볼 수 있음
SELECT * FROM USER_OBJECTS WHERE OBJECT_NAME='TRG_EMP_UPD_SAL';

-- 트리거 삭제
DROP TRIGGER TRG_EMP_UPD_SAL;
```

- 트리거 삭제후, 다시 프로시저 생성했더니 생성되었다-!

<br><br>

### 3-2. PROCEDURE 예외처리 구문 추가하고 PRINT
- 상황 : EXECUTE CHANGE_SALARY(7369, 1234567); 실행시
- ORA-01438: 이 열에 대해 지정된 전체 자릿수보다 큰 값이 허용됩니다.
- 에러 발생

```sql
SET SERVEROUTPUT ON
CREATE OR REPLACE PROCEDURE CHANGE_SALARY(A_EMPNO IN NUMBER, A_SALARY NUMBER DEFAULT 2000)
AS
BEGIN
    UPDATE EMP
    SET SAL = A_SALARY
    WHERE EMPNO = A_EMPNO;
    COMMIT;
EXCEPTION
    WHEN VALUE_ERROR THEN
        DBMS_OUTPUT.PUT_LINE('VALUE_ERR => '||SQLERRM);
        NULL;
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('NO_DATA_ERR => '||SQLERRM);
        ROLLBACK;
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('OTHERS_ERR => '||SQLERRM);
        ROLLBACK;
END CHANGE_SALARY;
/
```

- EXCEPTION 구문 추가하여 다시 실행하여 에러메시지를 PRINT할 수 있다.

```sql
CREATE OR REPLACE PROCEDURE CHANGE_SALARY(A_EMPNO IN NUMBER, A_SALARY NUMBER DEFAULT 2000)
AS
    V_AVG_SALARY    EMP.SAL%TYPE;   -- 평균급여
    V_SALARY        EMP.SAL%TYPE;   -- 급여  
    V_MAX_SALARY    EMP.SAL%TYPE :=90000;   -- 급여상한액
    V_ADD_SALARY_RATE   NUMBER     := 0.02; -- 급여 보정비율
BEGIN
    SELECT AVG(SAL) INTO V_AVG_SALARY FROM EMP;
    IF A_SALARY < V_AVG_SALARY THEN -- 평균 이하인 경우 2% 추가 보정
        V_SALARY := A_SALARY + ROUND(A_SALARY + V_ADD_SALARY_RATE,1);
    ELSIF A_SALARY > 99999 THEN -- 급여 상한액 초과시 ERR (-99999)
        V_SALARY:=-99999;
    ELSIF A_SALARY IS NULL THEN -- NULL인경우 0으로 치환
        V_SALARY :=0;
    ELSE
        V_SALARY := A_SALARY;
    END IF;
    
    UPDATE EMP
    SET SAL = V_SALARY
    WHERE EMPNO = A_EMPNO;
    COMMIT;
    
EXCEPTION
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('OTHERS_ERR => '||SQLERRM);
        ROLLBACK;
END CHANGE_SALARY;
/


EXECUTE CHANGE_SALARY(7369,1234567);
```

<br><br>

### 3-3. PROCEDURE 예외처리 구문 추가하고 로그저장
#### 3-3-1. 예외처리 로그 저장 테이블 생성

```sql
CREATE TABLE EXCEPTION_LOG(
    LOG_DATE        VARCHAR2(8)     DEFAULT TO_CHAR(SYSDATE,'YYYYMMDD') --로그기록일자
    , LOG_TIME      VARCHAR2(6)     DEFAULT TO_CHAR(SYSDATE,'HH24MISS') --로그기록 시간
    , PROGRAM_NAME  VARCHAR2(100)   -- EXCEPTION발생 프로그램
    , ERROR_MESSAGE VARCHAR2(250)   -- EXCEPTION발생 MESSAGE
    , DESCRIPTION      VARCHAR2(250)  --비고사항
);
```

#### 3-3-2. 에러발생시 저장하는 프로시저 생성

```sql
CREATE OR REPLACE PROCEDURE CHANGE_SALARY(A_EMPNO IN NUMBER, A_SALARY NUMBER DEFAULT 2000)
AS
    V_ERROR_MESSAGE EXCEPTION_LOG.ERROR_MESSAGE%TYPE; -- 에러메시지
BEGIN
    UPDATE EMP
    SET SAL = A_SALARY
    WHERE EMPNO = A_EMPNO;
    COMMIT;
    
EXCEPTION
    WHEN OTHERS THEN
        ROLLBACK;
        BEGIN
            V_ERROR_MESSAGE := SQLERRM;
            INSERT INTO EXCEPTION_LOG(PROGRAM_NAME, ERROR_MESSAGE, DESCRIPTION)
                VALUES( 'CHANGE_SALARY', V_ERROR_MESSAGE, 'VALUES : [1]=>'||A_EMPNO||' [2]=>'||A_SALARY);
            COMMIT;
        EXCEPTION
            WHEN OTHERS THEN
                NULL;
        END;
END CHANGE_SALARY;
/

EXECUTE CHANGE_SALARY(7369,1234567);
SELECT * FROM EXCEPTION_LOG;
```

- 결과

| LOG_DATE | LOG_TIME | PROGRAM_NAME | ERROR_MESSAGE | DESCRIPTION |
|:---:|:---:|:---:|---|---|
| 20210705 | 162055 | CHANGE_SALARY | ORA-01438: 이 열에 대해 지정된 전체 자릿수보다 큰 값이 허용됩니다. | VALUES : [1]=>7369 [2]=>1234567 |

<br><br>

### 3-3. PROCEDURE 예외처리 구문 로그테이블 추가부분 모듈화
- MODULE화 하여 모든 PL/SQL블록에서 공통적으로 사용할 수 있도록 한다.

#### 3-3-1. WRITE_LOG프로시저 생성

```sql
CREATE OR REPLACE PROCEDURE WRITE_LOG(A_PROGRAM_NAME IN VARCHAR2, A_ERROR_MESSAGE IN VARCHAR2, A_DESCRIPTION IN VARCHAR2)
AS
BEGIN
    INSERT INTO EXCEPTION_LOG(PROGRAM_NAME, ERROR_MESSAGE, DESCRIPTION)
        VALUES(A_PROGRAM_NAME, A_ERROR_MESSAGE, A_DESCRIPTION);
    COMMIT;
EXCEPTION
    WHEN OTHERS THEN
        NULL;
END;
/
```

#### 3-3-2. WRITE_LOG프로시저 사용

```sql
CREATE OR REPLACE PROCEDURE CHANGE_SALARY(A_EMPNO IN NUMBER, A_SALARY NUMBER DEFAULT 2000)
AS
    V_ERROR_MESSAGE EXCEPTION_LOG.ERROR_MESSAGE%TYPE;
BEGIN
    UPDATE EMP
    SET SAL = A_SALARY
    WHERE EMPNO = A_EMPNO;
    COMMIT;
    
EXCEPTION
    WHEN OTHERS THEN
        ROLLBACK;
        WRITE_LOG( 'CHANGE_SALARY', V_ERROR_MESSAGE, 'VALUES : [1]=>'||A_EMPNO||' [2]=>'||A_SALARY); -- 한줄로 재사용가능
END CHANGE_SALARY;
/

EXECUTE CHANGE_SALARY(7369,987654321);
SELECT EMPNO,SAL FROM EMP WHERE EMPNO=7369;
SELECT * FROM EXCEPTION_LOG;
```

<br><br><br><br>

