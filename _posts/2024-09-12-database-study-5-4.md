---
title: DB - 락(Lock)

author: milktea
date: 2024-09-12 10:00:00 +0800
categories: [Database]
tags: [Lock, ConcurrencyControl]
pin: true
math: true
mermaid: true
---

> 데이터베이스 스터디 5주차에서 학습하고 정리한 내용입니다.
{: .prompt-info }


# 1. 동시성 제어

동시성 제어는 여러 트랜잭션이 데이터를 읽고 조작할 때 데이터의 일관성과 무결성을 보장하기 위한 매커니즘이다.
이러한 동시성 제어에는 락, 트랜잭션 격리 수준, MVCC와 같은 여러 기법들이 있다.

## 갱신 손실 문제

트랜잭션 격리 수준 포스트에서 확인한 Dirty Read, Non-Repeatable Read, Phantom Read 문제는 모두 한 트랜잭션이 읽고 다른 트랜잭션이 쓰는 경우 발생하는 문제이다.
**두 트랜잭션이 동시에 쓰기를 시도하는 경우 갱신 손실 문제가 발생**한다.

한 이커머스 어플리케이션에서 회원이 상품을 주문하면 재고를 감소시키는 로직이 있다고 하자.

```sql
-- 1. 트랜잭션 시작
START TRANSACTION
-- 2. 주문한 만큼 재고를 감소하도록 수정
UPDATE products
SET stock = stock - /*주문한 수량*/
WHERE product_id = 1 AND stock > 0;
-- 3. 주문 내역을 order 테이블에 저장
INSERT INTO orders (...)
VALUES (...)
-- 4. 커밋
COMMIT
```

|   | 트랜잭션 A(1개 주문)                                                                                    | 트랜잭션 B(3개 주문)                                                                                    | 남은 수량 |
|---|---------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------|-----------|
| 1 | START TRANSACTION                                                                                       |                                                                                                         | 10        |
| 2 |                                                                                                         | START TRANSACTION                                                                                       | 10        |
| 3 | UPDATE products<br>SET stock = stock - 1<br>WHERE product_id = 1<br>AND stock > 0;<br>(변경한 재고는 9) |                                                                                                         | 10        |
| 4 |                                                                                                         | UPDATE products<br>SET stock = stock - 3<br>WHERE product_id = 1<br>AND stock > 0;<br>(변경한 재고는 7) | 10        |
| 5 | INSERT INTO orders (...)<br>VALUES (...)                                                                |                                                                                                         | 10        |
| 6 |                                                                                                         | INSERT INTO orders (...)<br>VALUES (...)                                                                | 10        |
| 7 | COMMIT<br>(DB에 9로 업데이트)                                                                           |                                                                                                         | 9         |
| 8 |                                                                                                         | COMMIT<br>(DB에 7로 업데이트)                                                                           | 7         |

트랜잭션 A와 트랜잭션 B가 거의 동시에 주문을 한 경우에는 두 트랜잭션 중 하나의 수정 결과가 덮어씌어워질 위험이 있다.
위와 같이 초기 수량이 10개인 상품을 두 명이 동시에 주문했을 때 A는 1개를 주문했고 B는 3개를 주문했으므로 6개가 남아야한다.
하지만 먼저 커밋된 트랜잭션 A의 결과가 반영되지 않아 DB 상의 재고는 7개가 남아있게 된다.

**이처럼 동시에 쓰기를 시도함으로써 발생하는 갱신 손실 문제는 데이터 정합성에 심각한 악영향을 미친다.**
실제 수량은 6이지만 DB에 7로 저장되어 있을 경우 실재고보다 초과하여 판매할 수 있는 위험이 있다.
이러한 갱신 손실을 해결하기 위해 락의 도입을 고려할 수 있다.

---
# 2. 락

락은 여러 커넥션에서 동일한 자원을 요청할 때 순서대로 하나의 커넥션에만 접근할 수 있게 해주는 기능이다.
위와 같이 두 트랜잭션이 동시에 하나의 재고에 접근하는 상황에서 A 트랜잭션이 내가 먼저 데이터를 변경하겠다는 락을 걸면 B 트랜잭션은 A의 데이터 변경 작업이 끝날 때까지 대기한다.
이렇게 **여러 커넥션에서 하나의 자원을 요청할 때 한번에 하나의 커넥션만 접근할 수 있도록 함으로써 쓰기 충돌로 인한 갱신 손실을 예방할 수 있다.**

MySQL은 MySQL 엔진에서 제공하는 락과 InnoDB 스토리지 엔진에서 제공하는 락이 있는데 위와 같은 갱신 손실 문제는 일반적으로 InnoDB 스토리지 엔진에서 제공하는 락으로 해결한다.
MySQL 엔진에서 제공하는 락은 테이블의 구조를 안정적으로 변경하거나, 데이터 백업을 목적으로 전체 데이터베이스를 잠그기 위해 사용되는 경우가 많다.

## 락의 종류

### 배타적 락(Exclusive Lock)

배타적 락은 다른 트랜잭션이 이 데이터에 대한 **쓰기 작업을 위한 배타적 락 획득과 읽기 작업을 위한 공유 락 획득까지 제한하는 것**이다.
참고로 **배타적 락이 걸려있어도 단순 SELECT 쿼리는 SERIALIZABLE 격리 수준이 아닌 한 아무런 락이 없기 때문에 읽기가 가능하다\![[출처]](https://velog.io/@soongjamm/Select-%EC%BF%BC%EB%A6%AC%EB%8A%94-S%EB%9D%BD%EC%9D%B4-%EC%95%84%EB%8B%88%EB%8B%A4.-X%EB%9D%BD%EA%B3%BC-S%EB%9D%BD%EC%9D%98-%EC%B0%A8%EC%9D%B4)**.

MySQL은 UPDATE, INSERT와 같은 수정 쿼리를 실행하면 자동으로 배타적 락을 얻고 또는 `SELECT ~ FOR UPDATE`로 수정할 레코드 또는 범위를 미리 선택하고 조회하면서 배타적 락을 얻을 수 있다.
`SELECT ~ FOR UPDATE`에 대한 설명은 [여기](https://milktea24.github.io/posts/database-study-2-4/#innodb%EC%9D%98-for-update-locking)에서 확인할 수 있다.

### 공유 락(Shared Lock)

공유 락은 다른 트랜잭션이 **이 데이터에 대한 공유 락 획득까지는 허용한다.**
다만 배타적 락 획득은 허용하지 않기 때문에 이 레코드에 대한 모든 공유 락이 해제될 때까지 다른 트랜잭션이 값을 수정할 수는 없다.
MySQL은 `SELECT ~ FOR SHARE`로 선택한 레코드 또는 범위에 대한 공유 락을 얻을 수 있다.

InnoDB에서 SERIALIZABLE 격리 수준이 아닌 한 단순 SELECT 쿼리는 공유 락을 얻지 않으므로 조회 트랜잭션에서 다른 트랜잭션이 데이터를 수정하는 상황을 막을려면 공유 락을 걸어야 한다.

## MySQL InnoDB의 락

InnoDB 스토리지 엔진은 레코드 기반의 잠금 기능을 제공한다.
일반적인 상용 DBMS와는 다르게 레코드와 레코드 사이의 간격을 잠궈 팬텀 읽기를 방지하는 갭 락도 함께 지원한다.
레코드 락, 갭 락, 넥스트 키 락에 대한 설명은 [여기](https://milktea24.github.io/posts/database-study-2-4/#innodb%EC%9D%98-for-update-locking)를 참고하자.

## MySQL InnoDB의 인덱스 기반 잠금

InnoDB는 인덱스를 통해 잠금을 설정하며 세컨더리 인덱스를 잠궈 기존 레코드의 수정 또는 조회를 막는다.
만약 세컨더리 인덱스가 없더라도 InnoDB는 레코드를 직접 잠그는 것이 아닌 클러스터링 인덱스를 잠근다.
따라서 InnoDB는 인덱스를 통해 간접적으로 레코드를 잠그는 것이다.

인덱스 포스트에서 특정 데이터를 조회할 때 인덱스 스캔을 수행하면서 해당하는 조건의 레코드가 있는지 찾는다고 설명했었다.
문제는 **REPEATABLE READ 격리 수준에서는 인덱스 스캔 범위 내에 팬텀 읽기가 발생하지 않도록 갭 락을 포함한 넥스트 락을 건다.**
그러므로 세컨더리 인덱스가 설정되지 않은 경우 InnoDB는 전체 클러스터링 인덱스를 스캔하는데 이 경우 테이블 내 전체 클러스터링 인덱스에 갭 락이 걸리게 된다.

반면 인덱스 레인지 스캔은 테이블 풀 스캔보다 범위가 적기 때문에 일부 데이터만 락을 걸어 동시 처리 성능이 향상된다.
따라서 **적절한 인덱스 설정은 조회 성능 뿐만 아니라 트랜잭션 동시성 제어 측면에서도 중요하다.**

READ COMMITTED 격리 수준에서는 갭 락을 사용하지 않기 때문에 인덱스를 스캔하면서 레코드 락을 걸고 스캔 후에는 바로 해제한다.

### 예시
```sql
-- REPEATABLE READ 격리 수준
-- 전체 직원은 10만 명
-- first_name이 Georgi인 직원은 250명, first_name이 Georgi이면서 last_name이 Klassen인 직원은 1명이다.

UPDATE employees SET hire_date=NOW() WHERE first_name='Georgi' AND last_name='Klassen';
```

- first_name 인덱스가 존재하는 경우

UPDATE를 할 데이터를 찾기 위해 인덱스 레인지 스캔을 활용한다.
이 때 first_name='Georgi'인 모든 데이터가 인덱스 레인지 스캔의 범위이므로 넥스트 락이 걸리는 인덱스의 수는 250개이다.
따라서 first_name이 'Georgi'가 아닌 직원 데이터의 업데이트를 수행하는 트랜잭션은 동시에 실행될 수 있다.

- 어떠한 세컨더리 인덱스도 없는 경우
 
UPDATE를 할 데이터를 찾기 위해 풀 테이블 스캔을 활용한다.
이 때 트랜잭션은 테이블 전체 직원에 넥스트 락을 걸어 수정하므로 락이 걸리는 인덱스의 수는 10만 개이다.
따라서 이 트랜잭션이 종료될 때까지 다른 모든 트랜잭션은 데이터를 삽입하거나 수정할 수 없다.


---
# Reference
[https://lsj31404.tistory.com/84](https://lsj31404.tistory.com/84)

[https://velog.io/@soongjamm/Select-%EC%BF%BC%EB%A6%AC%EB%8A%94-S%EB%9D%BD%EC%9D%B4-%EC%95%84%EB%8B%88%EB%8B%A4.-X%EB%9D%BD%EA%B3%BC-S%EB%9D%BD%EC%9D%98-%EC%B0%A8%EC%9D%B4](https://velog.io/@soongjamm/Select-%EC%BF%BC%EB%A6%AC%EB%8A%94-S%EB%9D%BD%EC%9D%B4-%EC%95%84%EB%8B%88%EB%8B%A4.-X%EB%9D%BD%EA%B3%BC-S%EB%9D%BD%EC%9D%98-%EC%B0%A8%EC%9D%B4)

백은빈, 이성욱 편저, Real MySQL 8.0, 위키북스
