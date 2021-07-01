---
title: PLSQL03_Block구조와 Exception
toc: true
toc_sticky: true
toc_label: "Contents of Page"

categories:
  - Study
tags:
  - SQL_Study
---

<br><br>

# Block 구조와Exception

## 1. Nested Block
- 중첩된 블록, 블록안에 또 다른 블록이 내포된 구조
- 예외처리, 모듈화를 위해 사용

- Nested_blk.sql

```sql
<<MAIN_BLK>>
DECLARE
  V_DNAME   VARCHAR2(14);
  V_DEPTNO  NUMBER(2);
  V_LOC     VARCHAR2(13);
BEGIN
  V_DEPTNO := 77;
  V_DNAME := 'GLOBAL_PART';
  V_LOC := 'MAIN_Blk';
  
  <<LOCAL_BLOCK1>>
  DECLARE
    V_DNAME   VARCHAR2(14);
    V_DEPTNO  NUMBER(2);
  BEGIN
    V_DEPTNO := 88;
    V_DNAME   := 'LOCAL_PART_1';
    V_LOC     := 'Nested_Blk1';
    INSERT INTO DEPT VALUES(V_DEPTNO, MAIN_BLK.V_DNAME, V_LOC);
  END LOCAL_BLOCK_1;
  
  <<LOCAL_BLOCK2>>
  DECLARE
    V_DNAME   VARCHAR2(14);
    V_DEPTNO  NUMBER(2);
  BEGIN
    V_DEPTNO := 99;
    V_DNAME   := 'LOCAL_PART_2';
    V_LOC     := 'Nested_Blk2';
    INSERT INTO DEPT VALUES(V_DEPTNO, MAIN_BLK.V_DNAME, V_LOC);
  END;
  
  INSERT INTO DEPT VALUES(V_DEPTNO, V_DNAME, V_LOC);
END MAIN_BLK;
/ 
```

- BLOCK안에서 COMMIT시 밖의 명령어도 COMMIT됨( COMMIT이 있는 곳이 END )
 



## 2. Nested Block과 Transaction
- Nested_Block_Trans.sql

```sql
INSERT INTO DEPT VALUES(66, 'OUTER_BLK_PART', 'Outlander');

<<MAIN_BLK>>
DECLARE
  V_DNAME   VARCHAR2(14);
  V_DEPTNO  NUMBER(2);
  V_LOC     VARCHAR2(13);
BEGIN
  V_DEPTNO := 77;
  V_DNAME := 'GLOBAL_PART';
  V_LOC := 'MAIN_Blk';
  
  <<LOCAL_BLOCK1>>
  DECLARE
    V_DNAME   VARCHAR2(14);
    V_DEPTNO  NUMBER(2);
  BEGIN
    V_DEPTNO := 88;
    V_DNAME   := 'LOCAL_PART_1';
    V_LOC     := 'Nested_Blk1';
    INSERT INTO DEPT VALUES(V_DEPTNO, MAIN_BLK.V_DNAME, V_LOC);
    COMMIT;
  END LOCAL_BLOCK_1;
  
  <<LOCAL_BLOCK2>>
  DECLARE
    V_DNAME   VARCHAR2(14);
    V_DEPTNO  NUMBER(2);
  BEGIN
    V_DEPTNO := 99;
    V_DNAME   := 'LOCAL_PART_2';
    V_LOC     := 'Nested_Blk2';
    INSERT INTO DEPT VALUES(V_DEPTNO, MAIN_BLK.V_DNAME, V_LOC);
    COMMIT;
  END LOCAL_BLOCK_2;
  
  INSERT INTO DEPT VALUES(V_DEPTNO, V_DNAME, V_LOC); 
  -- COMMIT이 없으므로, 블록은 끝났지만 트랜잭션은 끝난게 아님.
  
END;
/

SELECT * FROM DEPT;
DELETE FROM DEPT WHERE DEPTNO IN (66,77,88,99);
COMMIT;
```

- MAIN_BLK.V_DNAME : 메인블록의 변수 참조.
- BLOCK안에 실행할 변수가 없으면 바깥의 변수를 자동으로 참조한다.
- 결과
  * | DEPTNO | DNAME | LOC |
    |:---:|:---:|:---:|
    | 77 | GLOBAL_PART | Nested_Blk2 |
    | 88 | GLOBAL_PART | Nested_Blk1 |
    | 99 | GLOBAL_PART | Nested_Blk2 |
    
<br>


## 3. Exception 정의
## 4. Exception 발생 및 처리
