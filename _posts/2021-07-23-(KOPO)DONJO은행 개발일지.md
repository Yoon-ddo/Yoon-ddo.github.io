---
title: (KOPO)DONJO은행 개발일지
toc: true
toc_sticky: true
toc_label: "Contents of Page"
categories:
  - Project
tags:
  - Java_Project
---

<br><br>


# 문제1. SQL 프로시저 JAVA (DAO) 실행.
## 1. 상황
* TRANSACTION이라는 SQL프로시저를 생성했다.
* JAVA DAO에서 실행하려고하니까 부적절한 SQL문이라는 에러 등장.
* 우선 TRANSACTION 프로시저 코드에는 에러가 없었다.

```sql
CREATE OR REPLACE PROCEDURE TRANSACTION(
    V_MYACC           TRANHISTORY.MYACC%TYPE,
    V_MYBANK          TRANHISTORY.MYBANK%TYPE,
    V_INPUTBALANCE    TRANHISTORY.BALANCE%TYPE, -- 입금금액
    V_OTHACC          TRANHISTORY.OTHACC%TYPE, --상대계좌
    V_OTHBANK         TRANHISTORY.OTHBANK%TYPE --상대 은행
) IS
    V_TRANSEQ         TRANHISTORY.TRANSEQ%TYPE         := SEQ_TRANHISTORY_CODE.nextval;
   
BEGIN
    --계좌에 반영(내계좌)
    UPDATE ACCOUNTDB SET BALANCE = BALANCE - V_INPUTBALANCE WHERE ACCOUNT = V_MYACC AND BANK_CODE = V_MYBANK;
    -- 계좌에 반영( 상대계좌 )
    UPDATE ACCOUNTDB SET BALANCE = BALANCE + V_INPUTBALANCE WHERE ACCOUNT = V_OTHACC AND BANK_CODE = V_OTHBANK;
    
    -- 로그저장 (내계좌)
    INSERT INTO TRANHISTORY( TRANSEQ, MYACC, MYBANK, TR_CODE, BALANCE, OTHACC, OTHBANK)
        VALUES ( V_TRANSEQ, V_MYACC, V_MYBANK, 'M', V_INPUTBALANCE, V_OTHACC, V_OTHBANK );
    INSERT INTO TRANHISTORY( TRANSEQ, MYACC, MYBANK, TR_CODE, BALANCE, OTHACC, OTHBANK)
        VALUES ( V_TRANSEQ, V_OTHACC, V_OTHBANK, 'A', V_INPUTBALANCE, V_MYACC, V_MYBANK );
        
        
    COMMIT;
    
    EXCEPTION
        WHEN OTHERS THEN
        rollback;
        
 END;
/
EXEC TRANSACTION('35390201147037','D',7000,'33333333333338','D');
```

 ### 해결1
* JAVA DAO클래스에서 처음에 PreparedStatement로 sql을 실행했는데 주위에 자문을 구해보니
* `CallableStatement`를 사용해야 한다는 조언을 얻었다.
* 또한, SQL에서 exec 트랜잭션명 으로 실행하는 것과는 달리 `{ call 트랜잭션명() }`로 적어야 했다.

```java
/**
	 * 거래로그 및 잔액 업데이트 함수
	 * @param tran
	 * @throws SQLException
	 */
	public void transaction(TransactionVO tran) throws SQLException{
		
		
		Connection conn = new ConnectionFactory().getConnection(); 
                      //이건 미리 만들어둔 DB접속 패키지 함수
                      
		CallableStatement cstmt = null;
		String sql;
		
		try{
			
			sql= "{ call transaction(?,?,?,?,?) }";
			cstmt = conn.prepareCall(sql);
			
			int loc = 1;
			cstmt.setString(loc++, tran.getMyAcc()); // 내계좌
			cstmt.setString(loc++, tran.getMyBank()); // 내은행코드
			cstmt.setInt(loc++, tran.getBalance()); // 넘길금액
			cstmt.setString(loc++, tran.getOthAcc()); // 상대계좌
			cstmt.setString(loc++, tran.getOthBank()); // 상대 은행
			
			int result = cstmt.executeUpdate();
			System.out.println(result);
			cstmt.close();
			
		}catch (Exception e) {
			e.printStackTrace();
		}
	}
  ```
  
  * 오류메시지는 사라졌는데 DB에 변화가 전혀없었다. 뭘까...


 ### 해결2
 * Parameter값을 받아오면서 오탈자가 있었다... ㅋㅋㅋㅋ
 * 정말 두눈 크게 뜨고 개발해야지 ㅠㅠ

<br><br><br>

# 문제2. request영역에 setAttribute한 json리스트 ajax에서 불러오기
## 1. 상황
* Java Controller에서 request.setAttribute("transInfo", transInfo) 해준 데이터를
* 전혀 불러들이지 못하는 문제발생
