---
title: KDT:::Sparta MSA 아키텍처 3기(week07)
toc: true
toc_sticky: true
toc_label: "Contents of Page"
categories:
  - Study
tags:
  - MSA_architecture
---

<br><br><br>

# 1. 트랜잭션 어노테이션 동작 원리
* `@Transaction` : 여러개의 작업을 하나의 논리적 단위로 묶어 처리
* 커넥션연결 : 물리적연결 (host, port ...)
* 세션을 오픈 : 실질적 소통창구 ( 한개의 커넥션이 여러개의 세션을 가질 수도 있음 )
* 오픈한 세션에 auto commit = false로 변경
* 쿼리 실행 - 즉시 반영 되지 않음 (auto commit = false라서)
* commit / rollback 을 명시적으로 호출해야 반영된다.

<br><br>

# 2. JDBC 트랜잭션 코드
* 내부적으로 connection 선언해서 쓰는방법

```Java
  private final DataSource dataSource;

  public Category save(Category category) throws SQLException {
    String sql = "INSERT INTO category (name) VALUES (?);";

    Connection connection = null;
    PreparedStatement preparedStatement = null;
    try {
      connection = dataSource.getConnection(); // 이미 만들어진 커넥션 획득 (커넥션 pool에서)

      preparedStatement = connection.prepareStatement(sql);
      preparedStatement.setString(1, category.getName()); //📌 쿼리 (?) 부분 치환
      preparedStatement.executeUpdate(); // 📌 쿼리 실행

      return category;
    } catch (SQLException error) {
      log.error("insert : ", error);
      throw error;

    } finally {
      // 커넥션 해제 (안하면 다시 getConnection시 메모리 누수발생. 아직 점유중인걸로 판단함)
      JdbcUtils.closeStatement(preparedStatement);
      JdbcUtils.closeConnection(connection);
    }
```

* connection을 외부로부터 받아쓰는 방법

```Java
 public Category findById(Connection connection, Long categoryId) throws SQLException {
    String sql = "SELECT * FROM category WHERE id = ?";
    PreparedStatement pstmt = null;
    ResultSet rs = null;
    try {
      pstmt = connection.prepareStatement(sql);
      pstmt.setLong(1, categoryId);
      rs = pstmt.executeQuery();

      if (rs.next()) {
        return Category.builder()
            .name(rs.getString("name"))
            .build();

      } else {
        throw new NoSuchElementException("categoryId : " + categoryId);
      }
    } catch (SQLException error) {
      log.error("findById : ", error);
      throw error;

    } finally {
      JdbcUtils.closeResultSet(rs);
      JdbcUtils.closeStatement(pstmt);
    }
  }
```

* JDBC 관리

```Java
@Slf4j
@Service
@RequiredArgsConstructor
public class CategoryJdbcService {

  private final DataSource dataSource;
  private final CategoryJdbcRepository categoryJdbcRepository;

  public void update(Long categoryId, String name) throws SQLException {
    Connection connection = dataSource.getConnection();

    try {
      connection.setAutoCommit(false);

      Category category = categoryJdbcRepository.findById(connection, categoryId);

      if (Objects.nonNull(category)) {
        categoryJdbcRepository.update(connection, categoryId, name);
      }

      connection.commit();

    } catch (Exception error) {
      connection.rollback();
      throw new IllegalStateException(error);

    } finally {
      release(connection);
    }

  }

// 📌 초기화를 해줘야만함
  private void release(Connection connection) {
    if (connection != null) {
      try {
        connection.setAutoCommit(true);
        connection.close();
      } catch (Exception error) {
        log.error("release : ", error);
      }
    }
  }
}
```

<br>

* JPA에서 트랜잭션 관리하는 방법
  - 알아서 커넥션이 초기화되고, 관리가됨. (JDBC와의 차이점) 

```Java
@Slf4j
@Service
@RequiredArgsConstructor
public class ProductTransactionService {

  private final PlatformTransactionManager transactionManager;
  private final ProductRepository productRepository;

  public void updateProductStock(Long productId, int quantity) {
    TransactionStatus status = transactionManager.getTransaction(
        new DefaultTransactionDefinition());

    try {
      Product product = productRepository.findById(productId)
          .orElseThrow(() -> new RuntimeException("Product not found"));

      if (product.getStock() < quantity) {
        throw new IllegalArgumentException("Insufficient stock");
      }

      product.reduceStock(quantity);
      productRepository.save(product);

      log.info("isTransaction : {}", TransactionSynchronizationManager.isActualTransactionActive());

      transactionManager.commit(status);

    } catch (Exception ex) {
      transactionManager.rollback(status);
      throw ex;
    }
}
```
<br><br>

# 3. 격리성
* 동시성 문제를 얼마나 허용할지 설정
* `@Transactional(isolation = Isolation.READ_COMMITTED)`

| 값                  | 설명          |
| ------------------ | ----------- |
| `DEFAULT`          | DB 기본값      |
| `READ_UNCOMMITTED` | 더티 리드 허용    |
| `READ_COMMITTED`   | 커밋된 데이터만    |
| `REPEATABLE_READ`  | 동일 조회 결과 보장. 한 트랜잭션이 시작되기 전에 커밋된 내용만 조회하며, 트랜잭션이 끝날 때까지 다른 트랜잭션이 특정 데이터를 수정(UPDATE)하거나 삭제(DELETE)할 수 없도록 막습니다. 이로써 Non-Repeatable Read 문제를 해결합니다. |
| `SERIALIZABLE`     | 가장 엄격. 조회해온 목록을 gap lock 걸고, 다른 쪽에서 변경거래 시 막아버림. 특정 범위의 데이터를 읽을 때, 해당 범위 전체에 잠금을 걸어 다른 트랜잭션이 그 범위에 새로운 데이터를 추가(INSERT)하는 것조차 막습니다.      |

<br><br>

# 4. Dirty read
* commit되지 않은 데이터를 조회해옴.
* 격리레벨이 READ_UNCOMMITTED면 dirty read가 발생할 수 있다. 이때 격리레벨을 READ_COMMITTED로 올리면 해결가능
* READ_COMMITTED 시 Non-Repeatable Read(하나의 트랜잭션 내에서 두 번의 읽기 작업이 발생하는 상황) 발생가능
  - 리포트/정산 조회처럼 “트랜잭션 전체에서 같은 화면/스냅샷”이 중요 → 격리수준 올리기(가능하면 readOnly로)
  - 조회 후 반드시 이어서 수정(쿠폰 사용, 잔액 차감, 상태 변경 등) → SELECT FOR UPDATE(비관적 락) 또는 “조건부 UPDATE”
  - 락 경합이 걱정 + 업데이트 충돌만 감지하면 됨 → 낙관적 락(@Version)

<br><br>

# 5. Phantom Read
* 하나의 트랜잭션 안에서 같은 조건으로 두 번 조회했을 때, 없던 행이 생기거나(또는 있던 행이 사라지는) 현상
* 값이 바뀌는 게 아니라, 행의 개수/구성이 바뀜
* Non-Repeatable Read와 비교

| 구분    | Non-Repeatable Read | Phantom Read     |
| ----- | ------------------- | ---------------- |
| 변화 대상 | **기존 행의 값**         | **행의 개수/존재**     |
| 원인    | UPDATE              | INSERT / DELETE  |
| 예시    | 잔액 100 → 80         | ACTIVE 10건 → 11건 |

<br><br>

# 6. lock
* 종류
  - **레코드 잠금 (Record Lock)**: **특정 하나의 행(Row)**에만 거는 잠금입니다. `UPDATE`나 `DELETE` 시에 사용되어 Non-Repeatable Read를 방지하는 데 도움을 줍니다.
  - **갭 잠금 (Gap Lock)**: 데이터 행과 행 사이의 '**간격**'에 거는 잠금입니다. 이 간격에 새로운 데이터가 `INSERT`되는 것을 막는 역할을 합니다.
  - **넥스트 키 잠금 (Next-Key Lock)**: **레코드 잠금과 갭 잠금을 합친 것**으로, 특정 행과 그 이전 간격까지 함께 잠급니다. (MySQL InnoDB의 `REPEATABLE READ`에서 사용됨)
  - **범위 잠금 / 서술 잠금 (Range Lock / Predicate Lock)**: `WHERE` 절에 사용된 **조건(범위) 전체**를 잠그는 가장 강력한 잠금입니다.

## 6-1. 비관적 락(Pessimistic Lock)
* 비관적 락은 **데이터에 접근하기 전에 해당 자원을 선점하여 다른 트랜잭션이 접근하지 못하도록 차단하는 방식**입니다.
* 데이터를 보호하는 데 초점을 맞추며, 충돌 가능성이 높은 환경에서 주로 사용됩니다.
* default로 사용. 

## 6-2. 낙관적 락(Optimistic Lock)
* 낙관적 락은 데이터에 락을 걸지 않고 트랜잭션을 수행한 뒤, 충돌 여부를 확인하여 필요한 경우 작업을 다시 수행하는 방식입니다.
* 충돌이 드물다고 가정하는 환경에서 효율적으로 작동합니다.
* 지양하지만, 동시성 성능이 좋음


