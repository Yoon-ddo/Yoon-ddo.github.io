---
title: KDT:::Sparta MSA 아키텍처 3기(week03) 
toc: true
toc_sticky: true
toc_label: "Contents of Page"
categories:
  - Study
tags:
  - MSA_architecture
---

<br><br><br>

# 1. Spring AOP
## 1-1. AOP의 개념과 필요성 – 왜 써야 할까?
* 서비스 코드를 짜다 보면 여러 곳에 똑같이 들어가는 코드들이 있어요.
  - 로그 찍기 (요청 들어왔는지, 어떤 파라미터인지…)
  - 트랜잭션 시작/커밋/롤백
  - 권한 체크
  - 실행 시간 측정
  - 예외 공통 처리
* 문제점:
  - 코드 중복 증가
  - 비즈니스 로직 가독성 저하
  - 유지보수 어려움
* AOP는 이러한 공통 기능을 하나의 모듈(Aspect)에 모아서 관리하는 기술이다.
## 1-2. 해결책: 관심사의 분리
* 핵심 로직은 순수하게 유지
* 공통 기능은 Aspect에 모아 관리
* 효과
  - 코드 품질 향상
  - 유지보수 개선
  - 중복 제거 (DRY 원칙)
 
<br>

# 2. AOP 주요 개념
## 2-1. Aspect (관점)
* Aspect는 공통 기능을 모아둔 클래스이다.
```Java
@Aspect
@Component
public class LoggingAspect { }
```

## 2-2. Advice (조언)
* Advice는 언제 무엇을 실행하는지 정의한 메서드이다.
* 종류
  - @Before : 메서드 실행 전에 할 일
  - @AfterReturning : 정상 종료 후에 할 일
  - @AfterThrowing : 예외 발생 시 할 일
  - @After : 종료 후(성공/실패 상관 없이) 할 일
  - @Around : 앞뒤를 감싸서 전/후/예외까지 직접 제어

## 2-3. JoinPoint
* Advice가 실행될 수 있는 지점이다.
* Spring AOP에서는 메서드 실행 시점만 JoinPoint로 사용한다.
* JoinPoint 객체로 다음 정보들을 알 수 있어요.
  - 호출된 클래스 / 메서드 이름
  - 전달된 파라미터
  - 리턴 타입 등

## 2-4. Pointcut
* Advice를 적용할 대상을 선별하는 필터 역할이다.
* 수많은 JoinPoint 중 ‘어디에 적용할지’ 고르는 필터
* `execution([수식어] 리턴타입 [클래스경로.]메서드명(파라미터))`
```Java
@Pointcut("execution(* com.example.user..*(..))")
public void userMethods() {}
```

## 2-5. Pointcut 재사용
* 같은 조건(예: com.example.user..*)을 여러 Advice에서 쓰고 싶을 때
* 여러 Pointcut을 조합할 때
```Java
@Pointcut("execution(* com.example.user..*(..))")
public void userLayer() {}

@Pointcut("@annotation(LogExecutionTime)")
public void logTimeAnnotated() {}

@Pointcut("userLayer() && logTimeAnnotated()")
public void combined() {}
```

<br>

# 3. 리팩토링
* 애플리케이션의 겉으로 보이는 동작은 그대로 유지한 채, 코드의 내부 구조를 개선하는 체계적인 과정입니다.
* 단순히 코드를 정리하는 것을 넘어, 코드의 가독성, 유지보수성, 확장성을 극대화하여 장기적으로 소프트웨어의 가치를 높이는 엔지니어링 활동입니다.

## 3-1. 리팩토리 전 후비교
* Before
```Java
// 구매 처리, 유효성 검사, DB 저장을 모두 수행하는 거대한 메서드
public class PurchaseManager {

    public void processPurchase(Purchase purchase) {
        // 1. 구매 유효성 검사
        if (purchase.getPurchaseId() == null || purchase.getItems().isEmpty()) {
            throw new IllegalArgumentException("잘못된 구매 정보입니다.");
        }

        // 2. 재고 확인
        for (Item item : purchase.getItems()) {
            if (inventory.getStock(item) <= 0) {
                throw new RuntimeException("재고가 부족합니다.");
            }
        }

        // 3. 데이터베이스에 구매 정보 저장
        dbConnection.save(purchase);

        // 4. 이메일 발송
        emailSender.send(purchase.getCustomerEmail(), "구매가 완료되었습니다.");
    }
    
}
```

* After
```Java
public class PurchaseService {
		
		// 역할을 위임하여 로직을 명확하게 만듦
    private final PurchaseValidator validator;
    private final PurchaseRepository repository;
    private final NotificationService notificationService;
    
}
// 각 클래스는 자신의 책임만 수행한다.
class PurchaseValidator { /* 유효성 검사 책임 */ }
class PurchaseRepository { /* 데이터베이스 처리 책임 */ }
class NotificationService { /* 알림 발송 책임 */ }
```

