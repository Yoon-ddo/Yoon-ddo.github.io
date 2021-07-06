---
title: PLSQL08_Module(Stored Block3)
toc: true
toc_sticky: true
toc_label: "Contents of Page"

categories:
  - Study
tags:
  - SQL_Study
---

<br><br>

# PACKAGE
## 1. PACKAGE 만들기
### 1-1. HEADER만들기
- HEADER영역에서 정의한 PROCEDURE, FUNCTION, VARIABLE은 `PUBLIC`이다.

```sql
CREATE OR REPLACE PACKAGE P_EMPLOYEE
AS
    -- 사원 퇴사
    PROCEDURE DELETE_EMP(P_EMPNO EMP.EMPNO%TYPE);
    
    -- 신규 입사사원 등록
    PROCEDURE INSERT_EMP(P_EMPNO NUMBER, P_ENAME VARCHAR2, P_JOB VARCHAR2, P_SAL NUMBER, P_DEPTNO NUMBER);
    
    -- MANAGER이름 리턴
    FUNCTION SEARCH_MNG(P_EMPNO EMP.EMPNO%TYPE) RETURN VARCHAR2;
    GV_ROWS NUMBER(6); -- public변수
END P_EMPLOYEE;
/
```  



### 1-2. BODY만들기
- BODY영역에서만 정의한 PROCEDURE, FUNCTION, VARIABLE은 `PRIVATE`이다.
- GV_ROWS는 참조가능하지만, V_ROWS는 에러 발생
- HEADER에 정의하면 PUBLIC이 된다. BODY에 정의되어있는데 HEADER에 없다면 그것은 PRIVATE!

```sql
CREATE OR REPLACE PACKAGE BODY P_EMPLOYEE
AS
    V_ENAME     EMP.ENAME%TYPE; -- PRIVATE
    V_ROWS      NUMBER(6);      -- PRIVATE
    
    FUNCTION PRVT_FUNC(P_NUM IN NUMBER) RETURN NUMBER IS
    BEGIN
        RETURN P_NUM;
    END PRVT_FUNC;
    
    -- 신규 입사사원 등록
    PROCEDURE INSERT_EMP(P_EMPNO NUMBER, P_ENAME VARCHAR2, P_JOB VARCHAR2, P_SAL NUMBER, P_DEPTNO NUMBER)
    IS
    BEGIN
        INSERT INTO EMP(EMPNO, ENAME, JOB, SAL, DEPTNO) VALUES(P_EMPNO, P_ENAME, P_JOB, P_SAL, P_DEPTNO);
    END INSERT_EMP;
    
    PROCEDURE DELETE_EMP(P_EMPNO EMP.EMPNO%TYPE) IS --PUBLIC PROCEDURE정의
    BEGIN
        DELETE FROM EMP WHERE EMPNO = P_EMPNO;
        COMMIT;
        --IMPLICIT CURSOR ATTRIBUTE, PUBLIC변수 참조
        GV_ROWS := GV_ROWS+SQL%ROWCOUNT;
        V_ROWS := PRVT_FUNC(GV_ROWS); -- PRIVATE FUNCTION 참조
    EXCEPTION
        WHEN OTHERS THEN
            ROLLBACK;
            WRITE_LOG('P_EMPLOYEE.DELETE', SQLERRM, 'VALUES : [EMPNO] => '||P_EMPNO);
    END DELETE_EMP;
    
    FUNCTION SEARCH_MNG(P_EMPNO EMP.EMPNO%TYPE) RETURN VARCHAR2 IS
        V_ENAME     EMP.ENAME%TYPE;
    BEGIN
        SELECT ENAME INTO V_ENAME FROM EMP
        WHERE EMPNO = (SELECT MGR FROM EMP WHERE EMPNO = P_EMPNO);
        RETURN V_ENAME;
    EXCEPTION
        WHEN NO_DATA_FOUND THEN
            V_ENAME := 'NO_DATA';
            RETURN V_ENAME;
        WHEN OTHERS THEN
            V_ENAME := 'SUBSTR(SQLERRM,1,12)';
            RETURN V_ENAME;
    END SEARCH_MNG;
BEGIN
    GV_ROWS := 0; -- 초기화
END P_EMPLOYEE;
/
```

<br><br><br><br>

# TRIGGER
- 선언적 무결성 제약사항(PK, UK, FK, NN, CHECK)은 정의만 하면 된다. 하지만 단순하다. 
- TRIGGER는 PL/SQL로 코딩해야하기 때문에 복잡하지만, 복잡한 비즈니스 로직을 구현
- 이벤트가 발생하면 자동으로 실행.

|| INSERT | DELETE | UPDATE |
|:---:|:---:|:---:|:---:|
| :NEW | O | X | O |
| :OLD | X | O | O |

## 1. 트리거 사용

```sql
CREATE OR REPLACE TRIGGER TRG_CHANGE_SAL
BEFORE UPDATE OF SAL ON EMP
FOR EACH ROW
BEGIN
    IF( :NEW.SAL >9000 ) THEN
        :NEW.SAL := 9000;
    END IF;
END;
/
-- Trigger TRG_CHANGE_SAL이(가) 컴파일되었습니다.


UPDATE EMP SET SAL=9500 WHERE EMPNO IN (7369);
SELECT * FROM EMP WHERE EMPNO IN (7369);
```

- EMPNO가 7369인 사원의 SAL값을 9500으로 UPDATE했지만, TRIGGER로 인해 9000이 들어감.

## 2. 트리거 특징
- 파라미터가 없고 ORACLE에 의해 자동으로 호출
- PREVENT INVALID TRANSACTION
- EVENT LOGGING : 트리거 작동을 로그에 남긴다.
- 트리거 안에는 COMMIT, ROLLBACK사용하지 않는다.( 방법은 있음 - 독립트랜잭션 )

## 3. 트리거 실습
### 3-0. 상황설명
- 신규 입사자 직군의 CLERK나 SALESMAN인 경우 노조에 자동가입된다.
- 퇴직시 퇴직자 명단에 항상 등록된다.
- 기존 급여 시스템의 기능적 오류로 인해 사원 급여 항목이 종종 마이너스로 바뀐다.
- 마이너스가 될 경우 원래 급여로 되될려야 한다.


### 3-1. 실습 전 테이블 생성
```sql
-- 퇴사자 테이블
CREATE TABLE RETIRED_EMP(
    EMPNO       NUMBER(4) NOT NULL,
    ENAME       VARCHAR2(10),
    JOB         VARCHAR2(9),
    RETIRED_DATE      DATE
);

-- 노조테이블
CREATE TABLE LABOR_UNION(
    EMPNO   NUMBER(4) NOT NULL,
    ENAME   VARCHAR2(10),
    JOB     VARCHAR2(9),
    ENROLL_DATE DATE
);
```

### 3-2. 트리거 생성

```sql
CREATE OR REPLACE TRIGGER TRG_EMP_CHANGE
BEFORE INSERT OR DELETE OR UPDATE OF SAL ON EMP
FOR EACH ROW
DECLARE
BEGIN
    --TRIGGER EVENT를 구분.(INSERTING, DELETING, UPDATING)
    --CLERK, SALSEMAN인 경우 노조명단등록
    IF INSERTING AND :NEW.JOB IN('CLERK', 'SALESMAN') THEN
        INSERT INTO LABOR_UNION(EMPNO, ENAME, JOB, ENROLL_DATE) VALUES(:NEW.EMPNO, :NEW.ENAME, :NEW.JOB, SYSDATE);
        
    -- 퇴사시 퇴직자 명단에 등록
    ELSIF DELETING THEN
        BEGIN INSERT INTO RETIRED_EMP(EMPNO, ENAME, JOB, RETIRED_DATE)
            VALUES(:OLD.EMPNO, :OLD.ENAME, :OLD.JOB, SYSDATE);
            DELETE FROM LABOR_UNION WHERE EMPNO= :OLD.EMPNO;
        EXCEPTION
            WHEN OTHERS THEN
                NULL;
        END;
    ELSIF UPDATING THEN
        IF :NEW.SAL < 0 THEN -- 마이너스 급여가 될 경우 원래 급여로 치환
            :NEW.SAL := :OLD.SAL;
        END IF;
    END IF;
END;
/
```

### 3-3. 패키지와 함께 실습하기
- 우리는 1-2에서 이미 P_EMPLOYEE패키지를 생성했다!


```sql
BEGIN
    P_EMPLOYEE.DELETE_EMP(7369); -- 7369직원 퇴사
    P_EMPLOYEE.INSERT_EMP(2025,'JANG', 'PRESIDENT', 8888,10); -- PRESIDENT 입사
    P_EMPLOYEE.INSERT_EMP(10, 'KIM','SALESMAN', 5555, 10); -- SALESMAN입사
END;
/

-- TRIGGER 실습 1 ) 마이너스 급여업데이트시 기존급여로 치환
UPDATE EMP SET SAL = -1000 WHERE EMPNO = 7900;
-- TRIGGER 실습 1-1 ) 마이너스 급여는 기존급여로 치환되어있는지 확인
SELECT EMPNO, SAL FROM EMP WHERE EMPNO = 7900;

-- TRIGGER 실습 2 ) 퇴사직원 퇴사명단에 저장되어있는지 확인
SELECT * FROM RETIRED_EMP WHERE EMPNO = 7369;

-- TRIGGER 실습 3 ) SALESMAN직무 노조 가입되어있는지 확인
SELECT * FROM LABOR_UNION WHERE EMPNO IN(2025,10);
```
<br><br><br><br>
