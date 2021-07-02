---
title: PLSQL05_CURSOR
toc: true
toc_sticky: true
toc_label: "Contents of Page"

categories:
  - Study
tags:
  - SQL_Study
---

<br><br>

# CURSOR
## 1. 개념
- 사용자(User)가 SQL을 전송하면 Oracle DBMS는 해당 SQL을 실행하기 위한 메모리 공간이 필요하다.
  * Rows
  * Shared Pool에 저장된 Parsed Query에 대한 포인터
- Context Area : 할당된 영역, Cursor는 해당 영역을 가르키는 포인터나 해당 영역을 제어하기 위한 핸들러의 의미로 사용됨


<br><br><br><br>

## 2. 종류
### 2-1. 암시적 커서
- 자동적
- 실행부에서 사용되는 모든 커서는 암시적 커서
- PL/SQL 블록 내에서 사용되는 모든 DML 명령어와 SELECT는 암시적 커서를 통해 처리된다.
- 4단계로 처리
  * `DECLARE` -> `OPEN` -> `FETCH` -> `CLOSE`
- 암시적 커서는 개발자가 아닌 Oracle DBMS에서 자동으로 처리.

<br><br>

### 2-2. 명시적 커서
- 수동적으로
- 다중행을 조회하여 데이터를 처리하려할때 사용.

```sql
DECLARE
    CURSOR CUR_EMP IS
        SELECT EMPNO, JOB, SAL, COMM FROM EMP WHERE DEPTNO = 10;
        -- 부서번호가 10인 직원에 커서생성

    V_ENAME VARCHAR2(10);
    V_JOB   VARCHAR2(9);
    V_SAL   NUMBER(7,2);
    V_COMM  NUMBER(7,2);

BEGIN
    OPEN CUR_EMP;
    --커서 열기 : sql Execute한 후, ResultSet 생성.
    
    LOOP
        FETCH CUR_EMP INTO V_ENAME, V_JOB, V_SAL, V_COMM;
        --선언한 변수에 커서에 잡힌 데이터 야금야금 가져옴
        EXIT WHEN CUR_EMP%NOTFOUND;
        -- 다음 레코드가 없으면 loop빠져나감.
        
        INSERT INTO BONUS(ENAME, JOB, SAL, COMM)
            VALUES(V_ENAME, V_JOB, V_SAL, V_COMM);
            --db에 넣어줌
    END LOOP;
    
    DBMS_OUTPUT.PUT_LINE('TOTAL '||TO_CHAR(CUR_EMP%ROWCOUNT)||'rows precessed');
    --몇개 수행했는지
    
    CLOSE   CUR_EMP;
    --커서 닫기
    COMMIT;
END;
/
```

- `CUR_EMP%ROWCOUNT` : 처리된 행의 수 보여줌.
- commit을 LOOP문 안에 넣었을 때와 LOOP문 밖에두었을 때 성능차이는?
- JDBC,JSP로 해당 로직을 처리하는 것과 PLSQL을 사용했을 때의 성능차이는? 
  * PLSQL을 사용하는 것이 이득임!
- Fetch는 1Row단위로 수행한다. 1000만건의 대량 데이터를 다룰 때 어떻게 성능을 줄일까?
  * `Bulk Binding`을 사용하면 한번에 여러개를 빠르게 Fetch할 수 있다.
  * `Array Processing`, `Bulk Collect`

## 3. 커서 속성

| 커서속성자 | IMPLICIT CURSOR | EXPLICIT CURSOR |
|:---:|:---:|---|
| %ROWCOUNT | SQL문에의해 영향을 받은 ROW총 갯수 | FETCH된 누적 갯수 |
| %FOUND | SQL문에 의해 영향을 받은 ROW 존재유무<br>( TRUE / FALSE )리턴 | 현재 FETCH된 ROW 존재유무<br>( TRUE / FALSE )리턴 |
| %NOTFOUND | FOUND의 반대값 | FOUND의 반대값 |
| %ISOPEN | 항상 FALSE | CURSORDML Open상태 확인 |

### 3-1. 예제1

```sql
BEGIN
    DELETE FROM EMP WHERE SAL > 2000;
    DBMS_OUTPUT.PUT_LINE('[1-DELETE]'||TO_CHAR(SQL%ROWCOUNT)||'ROW IS DETETED');
    IF SQL%FOUND THEN
        DBMS_OUTPUT.PUT_LINE('SQL%FOUND = TRUE');
    ELSE
        DBMS_OUTPUT.PUT_LINE('SQL%NOTFOUND = FALSE');
    END IF;
    
    DBMS_OUTPUT.PUT_LINE('-------------------------------------------------------');
    
    DELETE FROM EMP WHERE SAL >2000;
    DBMS_OUTPUT.PUT_LINE('[2-DELETE]'||TO_CHAR(SQL%ROWCOUNT)||'ROW IS DELETED');
    
    IF SQL%FOUND THEN
        DBMS_OUTPUT.PUT_LINE('SQL%FOUND =TRUE');
    ELSE
        DBMS_OUTPUT.PUT_LINE('SQL%NOTFOUND = FALSE');
    END IF;
    ROLLBACK;
END;
/
```

```sql
------>[64초]
[1-DELETE]4ROW IS DETETED
SQL%FOUND = TRUE
-------------------------------------------------------
[2-DELETE]0ROW IS DELETED
SQL%NOTFOUND = FALSE


PL/SQL 프로시저가 성공적으로 완료되었습니다.
```

### 3-2. 예제2 : 커서 간단하게 정의하기

- 1단계

```sql
DECLARE
    CURSOR CUR_EMP IS
        SELECT ENAME, JOB, SAL, COMM FROM EMP WHERE DEPTNO = 10;
    R_CUR_EMP   CUR_EMP%ROWTYPE; -- 전체를 한 변수에 담아서 선언하기도 가능.
    
BEGIN
    OPEN CUR_EMP;
    LOOP
        FETCH CUR_EMP INTO R_CUR_EMP;
        
        EXIT WHEN CUR_EMP%NOTFOUND;
        
        INSERT INTO BONUS(ENAME, JOB, SAL, COMM)
            VALUES(R_CUR_EMP.ENAME, R_CUR_EMP.JOB, R_CUR_EMP.SAL, R_CUR_EMP.COMM);
    END LOOP;
    DBMS_OUTPUT.PUT_LINE('TOTAL '||TO_CHAR(CUR_EMP%ROWCOUNT)||'rows precessed');
    CLOSE   CUR_EMP;
    COMMIT;
END;
/
```

<br>

- 2단계
  * FOR LOOP에서는 OPEN, FETCH, CLOSE의 과정이 자동으로 이루어짐.
  * `DBMS_OUTPUT.PUT_LINE('TOTAL '||TO_CHAR(CUR_EMP%ROWCOUNT)||'rows precessed')`를 주석해제하면 안돌아간다!
    + 이유 : FOR문에서 커서가 자동으로 닫혔기 때문에 CUR_EMP%ROWCOUNT는 실행되지 않는다.

```sql
DECLARE
    CURSOR CUR_EMP IS
        SELECT ENAME, JOB, SAL, COMM FROM EMP WHERE DEPTNO = 10;
    
BEGIN
    FOR R_CUR_EMP IN CUR_EMP
    LOOP
        
        INSERT INTO BONUS(ENAME, JOB, SAL, COMM)
            VALUES(R_CUR_EMP.ENAME, R_CUR_EMP.JOB, R_CUR_EMP.SAL, R_CUR_EMP.COMM);
    END LOOP;
    --DBMS_OUTPUT.PUT_LINE('TOTAL '||TO_CHAR(CUR_EMP%ROWCOUNT)||'rows precessed');
    -- FOR문에서 커서가 자동으로 닫혔기 때문에 CUR_EMP%ROWCOUNT는 실행되지 않는다.
    COMMIT;
END;
/
```

<br><br><br><br>
