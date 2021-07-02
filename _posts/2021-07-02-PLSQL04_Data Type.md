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
### 3-1. RECORD
- C언어의 구조체와 동일한 개념
- 서로 다른 데이터형들을 논리적인 하나의 그룹으로 정의
- PL/SQL에서만 지원

```sql
SET SERVEROUTPUT ON;
DECLARE
    TYPE T_ADDRESS IS RECORD(
        ADDR1 VARCHAR2(60),
        ADDR2 VARCHAR2(60),
        ZIP   VARCHAR2(7),
        PHONE VARCHAR2(14)
    );
    
    TYPE T_EMP_RECORD IS RECORD(
        EMPNO NUMBER(4),
        ENAME VARCHAR2(10),
        JOB   VARCHAR2(9),
        ADDRESS T_ADDRESS,
        HIREDATE DATE
    );
    REC_EMP T_EMP_RECORD;
    
BEGIN
    --값 대입
    REC_EMP.EMPNO :=1234;
    REC_EMP.ENAME :='XMAN';
    REC_EMP.JOB   :='DBA';
    REC_EMP.ADDRESS.ADDR1 :='강남구 역삼동';
    REC_EMP.ADDRESS.ZIP :='150-036';
    REC_EMP.HIREDATE := SYSDATE -365;
    
    --조회
    DBMS_OUTPUT.PUT_LINE('********************************************************');
    DBMS_OUTPUT.PUT_LINE('사번 : '||REC_EMP.EMPNO);
    DBMS_OUTPUT.PUT_LINE('이름 : '||REC_EMP.ENAME);
    DBMS_OUTPUT.PUT_LINE('직업 : '||REC_EMP.JOB);
    DBMS_OUTPUT.PUT_LINE('주소 : '||REC_EMP.ADDRESS.ADDR1);
    DBMS_OUTPUT.PUT_LINE('주소 : '||REC_EMP.ADDRESS.ZIP);
    DBMS_OUTPUT.PUT_LINE('입사일 : '||TO_CHAR(REC_EMP.HIREDATE, 'YYYY/MM/DD'));
    DBMS_OUTPUT.PUT_LINE('********************************************************');
END;
/
```

- `T_ADDRESS` -> `T_EMP_RECORD` -> `REC_EMP`
  * 의 형식으로 참조하고있다.
  * REC_EMP는 T_EMP_RECORD 변수에에 접근할 수 있다.
  * T_EMP_RECORD의 ADDRESS는 T_ADDRESS 변수에 접근할 수 있다.
  * REC_EMP.ADDRESS.ADDR1, REC_EMP.ADDRESS.ZIP의 형태가 가능해짐.

<br><br>

### 3-2. INDES-BY TABLE
- C언어의 배열과 동일한 개념
- 동일 데이터 형을 하나의 그룹으로 정의
- PL/SQL에서만 지원

```sql
DECLARE
    TYPE T_EMP_LIST IS TABLE OF VARCHAR2(20)
        INDEX BY BINARY_INTEGER;
    TBL_EMP_LIST    T_EMP_LIST;
    V_TMP   VARCHAR2(20);
    V_INDEX NUMBER(10);
BEGIN
    TBL_EMP_LIST(1)     := 'SCOTT';
    TBL_EMP_LIST(1000)  := 'MILLER';
    TBL_EMP_LIST(-2134) := 'ALLEN';
    TBL_EMP_LIST(0)     := 'XMAN';
    
    V_TMP   := TBL_EMP_LIST(1000);
    
    --TABLE에 있는 DATA를 조회해서 RETURN한다.
    DBMS_OUTPUT.PUT_LINE('DATA OF KEY 1000 IS '||TBL_EMP_LIST(1000));
    DBMS_OUTPUT.PUT_LINE('DATA OF KEY -2134 IS '||TBL_EMP_LIST(-2134));
    DBMS_OUTPUT.PUT_LINE('DATA OF KEY 1 IS '||TBL_EMP_LIST(1));
    
    --METHOD사용 (DELETE, COUNT, FIRST, NEXT)
    IF NOT TBL_EMP_LIST.EXISTS(888) THEN
        DBMS_OUTPUT.PUT_LINE('DATA OF KEY 888 IS NOT EXIST');
    END IF;
    
    --LOOP를 사용하여 조회
    V_INDEX := TBL_EMP_LIST.FIRST; -- PRIOR, FIRST, LAST
    LOOP
        DBMS_OUTPUT.PUT_LINE('LOOP : '||TO_CHAR(V_INDEX)||' ==> '||TBL_EMP_LIST(V_INDEX));
        V_INDEX := TBL_EMP_LIST.NEXT(V_INDEX);
        EXIT WHEN V_INDEX IS NULL;
    END LOOP;
    
    DBMS_OUTPUT.PUT_LINE('DATA OF KEY 999 IS '||TBL_EMP_LIST(999));
    DBMS_OUTPUT.PUT_LINE('DATA OF KEY 0 IS '||TBL_EMP_LIST(0));
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('ERR CODE => '||TO_CHAR(SQLCODE));
        DBMS_OUTPUT.PUT_LINE('ERR MSG => '||SQLERRM);
END;
/
```

- 결과

```sql
PL/SQL 프로시저가 성공적으로 완료되었습니다.

DATA OF KEY 1000 IS MILLER
DATA OF KEY -2134 IS ALLEN
DATA OF KEY 1 IS SCOTT
DATA OF KEY 888 IS NOT EXIST
LOOP : -2134 ==> ALLEN
LOOP : 0 ==> XMAN
LOOP : 1 ==> SCOTT
LOOP : 1000 ==> MILLER
ERR CODE => 100
ERR MSG => ORA-01403: 데이터를 찾을 수 없습니다.

```

- `V_INDEX := TBL_EMP_LIST.FIRST;` 가장 첫번째 인덱스(작은값) 대입.

### RECORD와 TABLE타입을 함께 쓸 수 있을까?
- YES!
- 레코드와 테이블은 DBMS메모리 안에 SESSION 인근에 생성됨.

<br><br><br>

## 4. 참조연산
### 4-1. %TYPE
- `EMP.EMPNO%TYPE` : 특정 테이블, 컬럼의 데이터 타입과 LENGTH를 참조하여 변수 정의
- 장점
  * 편리성
  * 유연성 : 데이터 길이를 변경할 때 코드를 안적어도 된다.  

### 4-2. %ROWTYPE
- TABLE, VIEW, CURSOR의 여러 COLUMN을 참조하여 RECORD TYPE생성.
- `REC_EMP EMP%ROWTYPE` : EMP 테이블내의 모든 컬럼을 %TYPE으로 참조한 후, 내부적으로 RECORD로 정의됨.

```sql
DECLARE
    REC_EMP     EMP%ROWTYPE;
    V_EMPNO     EMP.EMPNO%TYPE;
BEGIN
    --1개의 ROW를 SELECT해서 RECORD에 저장.
    SELECT * INTO REC_EMP FROM EMP WHERE EMPNO = 7369;
    -- INTO 구문 필수
    
    DBMS_OUTPUT.PUT_LINE('EMPNO         => '||REC_EMP.EMPNO);
    DBMS_OUTPUT.PUT_LINE('ENAME         => '||REC_EMP.ENAME);
    DBMS_OUTPUT.PUT_LINE('JOB           => '||REC_EMP.JOB);
    DBMS_OUTPUT.PUT_LINE('MGR           => '||REC_EMP.MGR);
    DBMS_OUTPUT.PUT_LINE('HIREDATE      => '||REC_EMP.HIREDATE);
    DBMS_OUTPUT.PUT_LINE('SAL           => '||REC_EMP.SAL);
    
    --RECORD의 각각 FIELD를 독립적으로 사용가능
    SELECT EMPNO, ENAME INTO V_EMPNO, REC_EMP.ENAME
    FROM EMP
    WHERE EMPNO = 7369;
    
    DBMS_OUTPUT.PUT_LINE('---------------------------------');
    DBMS_OUTPUT.PUT_LINE('EMPNO         => '||V_EMPNO);
    DBMS_OUTPUT.PUT_LINE('ENAME         => '||REC_EMP.ENAME);
END;
/
```

<br><br><br>

## 5. Block내의 Select
### 5-1. 데이터가 없는 경우

```sql
ECLARE
    V_EMPNO     EMP.EMPNO%TYPE;
    V_ENAME     EMP.ENAME%TYPE;
    V_HIREDATE  EMP.HIREDATE%TYPE;
BEGIN
    -- SELECT 되는 대상 데이터가 없는 조회
    SELECT EMPNO, ENAME, HIREDATE INTO V_EMPNO, V_ENAME, V_HIREDATE
    FROM EMP
    WHERE EMPNO = 1;
    DBMS_OUTPUT.PUT_LINE('SELECTED EXACTLY ONE ROW '||V_EMPNO);
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('NO DATA FOUND!');
    WHEN TOO_MANY_ROWS THEN
        DBMS_OUTPUT.PUT_LINE('TOO MANU ROWS FOUND!');
END;
/
```

- EMPNO=1인 데이터는 존재하지 않는다.
- EXCEPTION구문에서 NO_DATA_FOUND 예외가 발생하여 해당 예외처리 수행
- `NO DATA FOUND!`
  * PL/SQL블록 내에서 SELECT한 데이터를 기반으로 다른 연산을 수행한다.
  * SELECT한 것이 없어서 후속 연산을 수행할 수 없어 ERROR로 간주하여 예외를 처리할 수 있도록 한 것이다. 

### 5-2. 데이터가 여러개인 경우

```sql
DECLARE
    V_EMPNO     EMP.EMPNO%TYPE;
    V_ENAME     EMP.ENAME%TYPE;
    V_HIREDATE  EMP.HIREDATE%TYPE;
BEGIN
    -- SELECT 되는 대상 데이터가 없는 조회
    SELECT EMPNO, ENAME, HIREDATE INTO V_EMPNO, V_ENAME, V_HIREDATE
    FROM EMP
    WHERE EMPNO >= 1;
    DBMS_OUTPUT.PUT_LINE('SELECTED EXACTLY ONE ROW '||V_EMPNO);
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('NO DATA FOUND!');
    WHEN TOO_MANY_ROWS THEN
        DBMS_OUTPUT.PUT_LINE('TOO MANU ROWS FOUND!');
END;
/
```

- EMPNO >= 1에 해당하는 데이터가 여러개이다.
- `SELECT EMPNO, ENAME, HIREDATE INTO V_EMPNO, V_ENAME, V_HIREDATE`에 데이터가 1개씩 들어가므로, TOO_MANY_ROWS 예외처리를 수행한다.
  * 스칼라 변수에 복수개의 값이 저장될 수 없다.
  * RUN TIME ERROR시 발생하는 에러 

<br><br><br><br>
