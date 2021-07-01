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

--ROLLBACK시  INSERT INTO DEPT VALUES(V_DEPTNO, V_DNAME, V_LOC); 취소됨

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
    
<br><br><br>

## 3. Exception 정의
* 컴파일 시점 에러
* 실행시점 에러
* PLSQL 블록은 컴파일 -> 실행 순으로 처리

### 3-1. 컴파일 시점 에러

```sql
SET SERVEROUTPUT ON
INSERT INTO DEPT VALUES(66,'OUTER_BLK_PART', 'Outlander');

DECLARE
  V_DNAME   VARCHAR2(14);
  --V_DEPTNO  NUMBER(200); --최대가 38이라서 200은 에러남. NUMBER 정도 제약은 (1 .. 38)범위이어야 합니다
  V_DEPTNO  NUMBER(2);
  V_LOC     VARCHAR2(13);

BEGIN 
  V_DEPTNO :=77;
  V_DNAME  :='GLOBAL_PART';
  V_LOC    :='Main_Blk';
  
  <<Nested_BLOCK_1>>
  DECLARE
    V_DNAME   VARCHAR2(14);
    V_DEPTNO  NUMBER(2);
  BEGIN
    V_DEPTNO  :=88;
    V_DNAME   :='LOCAL_PART_1';
    V_LOC     :='Nested_Blk1';
    
    INSERT INTO DEPT VALUES(V_DEPTNO, V_DNAME, V_LOC);
    COMMIT;
  END Nested_BLOCK_1;
  
  <<Nested_BLOCK_2>>
  DECLARE
    V_DNAME   VARCHAR2(14);
    V_DEPTNO  NUMBER(2);
  BEGIN
    V_DEPTNO  :=99;
    V_DNAME   :='LOCAL_PART_2';
    V_LOC     :='Nested_Blk2';
    
    INSERT INTO DEPT VALUES(V_DEPTNO, V_DNAME, V_LOC);
    COMMIT;
  END Nested_BLOCK_2;
  
  INSERT INTO DEPT VALUES(V_DEPTNO, V_DNAME, V_LOC);
END;
/

SELECT * FROM DEPT WHERE DEPTNO IN (66,77,88,99);
DELETE FROM DEPT WHERE DEPTNO IN (66,77,88,99);
COMMIT;  
```

### 3-2. 실행시점 에러

```sql
BEGIN
  INSERT INTO DEPT VALUES(66,'OUTER_BLK_PART', 'MAIN_BLK');
  
  <<Nested_BLOCK_1>>
  BEGIN
    INSERT INTO DEPT VALUES(76,'LOCAL_PART_1', 'Nested_Blk1');
    -- INSERT INTO DEPT VALUES(777,'LOCAL_PART_1', 'Nested_Blk1'); -- runtime err발생.
    -- NUMBER(2)라서 3자리는 '이 열에 대해 지정된 전체 자릿수보다 큰 값이 허용됩니다.' 에러 발생
    INSERT INTO DEPT VALUES(77,'LOCAL_PART_1', 'Nested_Blk1'); 
    INSERT INTO DEPT VALUES(78,'LOCAL_PART_1', 'Nested_Blk1');
    COMMIT;
   END Nested_BLOCK_1;
   
  <<Nested_BLOCK_2>>
  BEGIN
    INSERT INTO DEPT VALUES(88,'LOCAL_PART_2', 'Nested_Blk2');
    COMMIT;
  END Nested_BLOCK_2;
  INSERT INTO DEPT VALUES(99,'OUTER_BLK_PART', 'MAIN_BLK');
END;
/

SELECT * FROM DEPT WHERE DEPTNO IN (66,76,77,78,88,99);
DELETE FROM DEPT WHERE DEPTNO IN (66,76,77,78,88,99);
COMMIT;  
```

- Nested_BLOCK_1의 INSERT INTO DEPT VALUES(777,'LOCAL_PART_1', 'Nested_Blk1'); 구문 에러
  * 해당 구문은 NUMBER(2)라서 3자리는 '이 열에 대해 지정된 전체 자릿수보다 큰 값이 허용됩니다.' 에러 발생한다.
  * 블록안에서 에러 발생시 해당 블록의 예외처리 부분으로 넘어가는데 위의 코드는 EXCEPTION부분이 없다.
  * 그래서 바깥 블록의 예외처리 부분으로 넘어가게 되는데 바깥에도 없어서 에러가 발생되기 때문에 모든 블록은 실행되지 않게 된다.
  * Update 연산이 총 10개의 Row를 수정해야하는데 8번째에서 에러가 발생하면 1~8Row 까지 수정되었던 사항을 Rollback 처리를한다.
  * 해당 Statement내에 변경된 사항을 Rollback 하는 것이 Statement Level Rollback 이다.


- 예외처리부에서 EXCEPTION을 처리할 수 있는 경우
  * EXCEPTION을 처리한 후, 해당 BLOCK을 종료한 후 BLOCK의 다음 Statement 실행
- 예외처리부에서 EXCEPTION을 처리할 수 없는 경우
  * 예외처리부가 없는 경우
  * 예외처리부가 있지만 해당 EXCEPTION을 제어하지 못하는 경우
  * 해당 블록을 종료한 후, EXCEPTION을 외부에 전달.(Exception Propagation)

<br><br><br>

## 4. Exception 발생 및 처리
