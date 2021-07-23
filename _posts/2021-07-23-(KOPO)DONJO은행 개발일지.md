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

## 2. 원인
* 내가 ajax를 잘못이해하고 있었다.
* 계좌이체화면.jsp 에서 이미 가지고있는 본인 계좌를 불러오려면 바로 불러오는게 아니라 새로운 화면이 필요하다
* 계좌이체화면(전).jsp와 이를 부르는 controller가 필요하고 이것을 beans.properties에 등록해놔야함.
  - /accountTransferBefore.do=		kr.ac.donjo.account.controller.AccountTransferBeforeController 

## 3. 해결코드
* AccountTransferBeforeController.java

```java
package kr.ac.donjo.account.controller;

import java.util.List;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

import com.google.gson.Gson;

import kr.ac.donjo.account.dao.AccountInqueryDAO;
import kr.ac.donjo.account.vo.AccountVO;
import kr.ac.donjo.controller.Controller;
import kr.ac.donjo.login.vo.LoginVO;

public class AccountTransferBeforeController implements Controller {

	@Override
	public String handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
  
		AccountInqueryDAO dao = new AccountInqueryDAO();
		HttpSession session = request.getSession();
		LoginVO uservo = (LoginVO)session.getAttribute("userVO");
    // ID로 보유한 계좌 불러오는 부분.
		List<AccountVO> userAcc = dao.showAllAcc(uservo.getId());


		Gson gson = new Gson();// GSON사용하려면 jar파일 먼저 lib에 저장할것
		String transInfo = gson.toJson(userAcc);
		//System.out.println(transInfo);
		
		request.setAttribute("transInfo", transInfo);
		
		
		return "/jsp/account/account_transferbefore.jsp";
	}

}
```

* account_transferbefore.jsp
  - 아래처럼 찍어보면 json배열 값이 나옴.   

~~~html

<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>


${ transInfo }

  
~~~

* account_transfer.jsp (계좌이체하는 페이지)

```javascript
/**
	selection에 따라 값 변경

 */

$(document).ready(function(){
	

	
	$('select[name$=myBank]').change(function(){
		
		//입력받은 은행
		let bank = $('select[name$=myBank]').val()
		
		
		$.ajax({
			type : 'post',
			url  : '/Donjo-Web/accountTransferBefore.do',
		//	contentType : "application/json; charset=utf-8",
		//	dataType : 'json',

			success : callback,
			error : function(){
				alert('실패')
			}
		})
	})
})

function callback(data){
	//var userAcc = $(transInfo);
	console.log(data) // 웹 콘솔창에 찍어보면 json배열이 나옴.
}
```

* AJAX때문에 눈물 찔끔 날뻔했다..ㅠㅜ 열심히 공부해야지


