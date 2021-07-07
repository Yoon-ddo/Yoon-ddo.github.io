---
title: PLSQL09_Advanced Feature
toc: true
toc_sticky: true
toc_label: "Contents of Page"

categories:
  - Study
tags:
  - SQL_Study
---

<br><br>

# Dynamic SQL과 Static SQL
- Dynamic SQL : 실행시점 SQL생성
  * 개발자는 Dynamic SQL을 선호. 
- Static SQL : compile시점에 SQL생성(개발할때 만드는 SQL)
  * 관리자 입장에서 Static SQL을 선호함.
  * 성능적인 부분을 예측할 수 있기 때문
  * 보안적 측면도 좋음 (SQL Injection이 거의 불가능함.

## 1. Dynamic SQL
- DDL계열의 CREATE TABLE 사용시 에러발생

```sql
BEGIN
    CREATE TABLE BY_DYNAMIC(X DATE) -- DDL 계열의 명령어 사용시 에러발생.
END;
/
```

- PL/SQL 블록 내에서 DDL, DCL 등의 계열을 동적으로만 사용할 수 있음

```sql
DECLARE
    V_SQL       VARCHAR2(2000);
BEGIN
    BEGIN
        V_SQL := 'DROP TABLE BY_DYNAMIC';
            EXECUTE IMMEDIATE V_SQL;
        EXCEPTION
            WHEN OTHERS THEN
                DBMS_OUTPUT.PUT_LINE('DYNAMIC SQL DROP -> '||SUBSTR(SQLERRM, 1, 50));
    END;
    BEGIN
        EXECUTE IMMEDIATE 'CREATE TABLE BY_DYNAMIC(X DATE)';
    EXCEPTION
        WHEN OTHERS THEN
            DBMS_OUTPUT.PUT_LINE('DYNAMIC SQL CREATE -> '||SUBSTR(SQLERRM, 1, 50));
    END;
END;
/
```

<br><br><br>
## 2. Dynamic CURSOR
## 3. AUTONOMOUS TRANSACTION
## 4. BULK COLLECT & FORALL

