---
title: DB - SELECT와 응용(FOR UPDATE, GROUP BY, ORDER BY)
author: milktea
date: 2024-08-21 14:00:00 +0800
categories: [Database]
tags: [SQL]
pin: true
math: true
mermaid: true
---

> 데이터베이스 스터디 2주차에서 학습하고 정리한 내용입니다.
{: .prompt-info }

# 1. SELECT 절 처리 순서

- customers 테이블

| customer_id | customer_name |
|-------------|---------------|
| 1           | Kim           |
| 2           | Lee           |
| 3           | Park          |
| 4           | Jung          |

- orders 테이블

| order_id | customer_id | total_amount |
|----------|-------------|--------------|
| 1        | 1           | 8000         |
| 2        | 1           | 3000         |
| 3        | 2           | 8000         |
| 4        | 2           | 9000         |
| 5        | 3           | 7000         |
| 6        | 4           | 13000        |

```sql
SELECT 
    c.customer_name, 
    SUM(o.total_amount) AS total_spent
FROM 
    customers c
INNER JOIN 
    orders o ON c.customer_id = o.customer_id
WHERE 
    c.customer_name != 'Jung'
GROUP BY 
    c.customer_name
HAVING 
    SUM(o.total_amount) >= 10000
ORDER BY 
    total_spent DESC
LIMIT 
    5;
```
이 쿼리는 'Jung'고객을 제외하여 고객의 주문 총액을 계산하고 만원 이상인 고객들을 내림차순으로 정렬하는 기능을 가진 쿼리이다.
이 과정에서 상위 5명만 가져오는 페이징이 적용되어 있다.
이러한 SELECT 절은 다음과 같은 순서로 처리된다.

1. 데이터 조인
2. 행 필터링
3. 그룹화
4. 그룹 필터링
5. 반환할 데이터 선택
6. 정렬 및 페이징

## 1) 데이터 조인

관계 대수에서 조인은 두 테이블의 Cross Product한 후 특정한 조건을 적용하는 과정이다.
따라서 먼저 customers 테이블과 orders 테이블을 먼저 Cross Product를 수행한다.
아래 테이블은 Cross Product를 적용한 결과이다.

> DBMS에서는 최적화 등의 이유로 이론과 달리 Cross Product를 생성하지는 않는다. 대신 [Nested Loop Join](https://coding-factory.tistory.com/756#google_vignette) 등 여러 알고리즘을 사용한다. 아래의 Cross Product는 관계 대수 이론상 과정임을 참고하자.
{: .prompt-warning }

```sql
FROM customers c INNER JOIN orders o
```


| order_id | customer_id_order | total_amount | customer_id_customer | customer_name |
|----------|-------------------|--------------|----------------------|---------------|
| 1        | 1                 | 8000         | 1                    | Kim           |
| 1        | 1                 | 8000         | 2                    | Lee           |
| 1        | 1                 | 8000         | 3                    | Park          |
| 1        | 1                 | 8000         | 4                    | Jung          |
| 2        | 1                 | 3000         | 1                    | Kim           |
| 2        | 1                 | 3000         | 2                    | Lee           |
| 2        | 1                 | 3000         | 3                    | Park          |
| 2        | 1                 | 3000         | 4                    | Jung          |
| 3        | 2                 | 8000         | 1                    | Kim           |
| 3        | 2                 | 8000         | 2                    | Lee           |
| 3        | 2                 | 8000         | 3                    | Park          |
| 3        | 2                 | 8000         | 4                    | Jung          |
| 4        | 2                 | 9000         | 1                    | Kim           |
| 4        | 2                 | 9000         | 2                    | Lee           |
| 4        | 2                 | 9000         | 3                    | Park          |
| 4        | 2                 | 9000         | 4                    | Jung          |
| 5        | 3                 | 7000         | 1                    | Kim           |
| 5        | 3                 | 7000         | 2                    | Lee           |
| 5        | 3                 | 7000         | 3                    | Park          |
| 5        | 3                 | 7000         | 4                    | Jung          |
| 6        | 4                 | 13000        | 1                    | Kim           |
| 6        | 4                 | 13000        | 2                    | Lee           |
| 6        | 4                 | 13000        | 3                    | Park          |
| 6        | 4                 | 13000        | 4                    | Jung          |


On 조건을 처리하면서 customers 테이블의 customer_id와 orders 테이블의 customer_id와 일치하는 데이터만 남긴다.

```sql
orders o ON c.customer_id = o.customer_id
```

| order_id | customer_id | total_amount | customer_name |
|----------|-------------|--------------|---------------|
| 1        | 1           | 8000         | Kim           |
| 2        | 1           | 3000         | Kim           |
| 3        | 2           | 8000         | Lee           |
| 4        | 2           | 9000         | Lee           |
| 5        | 3           | 7000         | Park          |
| 6        | 4           | 13000        | Jung          |

## 2) 행 필터링

각 행을 평가하면서 조건에 부합하지 않으면 제거한다.

```sql
WHERE c.customer_name != 'Jung'
```

| order_id | customer_id | total_amount | customer_name |
|----------|-------------|--------------|---------------|
| 1        | 1           | 8000         | Kim           |
| 2        | 1           | 3000         | Kim           |
| 3        | 2           | 8000         | Lee           |
| 4        | 2           | 9000         | Lee           |
| 5        | 3           | 7000         | Park          |

## 3) 그룹화

행들을 그룹으로 묶는다.
이 경우 customer_name을 그룹으로 묶어 Select 실행 시 그룹 단위로 평가한다.

```sql
GROUP BY c.customer_name
```

## 4) 그룹 필터링

HAVING 절은 GROUP BY 뒤에 실행되어 행이 아닌 각각의 그룹마다 조건을 평가하여 해당하지 않는 조건은 제거한다.
'Park'의 경우 total_amount의 합이 10000을 넘지 않으므로 제외한다.

```sql
HAVING 
    SUM(o.total_amount) >= 10000
```

| order_id | customer_id | total_amount | customer_name |
|----------|-------------|--------------|---------------|
| 1        | 1           | 8000         | Kim           |
| 2        | 1           | 3000         | Kim           |
| 3        | 2           | 8000         | Lee           |
| 4        | 2           | 9000         | Lee           |

## 5) 반환할 데이터 선택

```sql
SELECT 
    c.customer_name, 
    SUM(o.total_amount) AS total_spent
```

| customer_name | total_spent |
|---------------|-------------|
| Kim           | 11000       |
| Lee           | 17000       |

이 과정에서 DISTINCT, MAX, SUM 등과 같은 함수를 적용한다.
또한 별칭도 함께 적용한다.

## 6) 정렬 및 페이징

쿼리 결과에 대한 정렬을 수행하고 결과를 페이징한다.

```sql
ORDER BY 
    total_spent DESC
LIMIT 
    5;
```

| customer_name | total_spent |
|---------------|-------------|
| Lee           | 17000       |
| Kim           | 11000       |

---
# 2. SELECT ~ FOR UPDATE 
`SELECT ~ FOR UPDATE` 키워드는 선택한 데이터에 배타적 잠금을 얻는다.
한 트랜잭션이 SELECT ~ FOR UPDATE를 선언하였다면 SELECT한 데이터는 수정이 필요하므로 다른 트랜잭션에서 수정하지 못하게 락을 얻겠다는 말이다.
이미 한 데이터를 어떤 트랜잭션이 락을 걸어두었다면 다른 트랜잭션은 이 데이터를 수정할 수 없다.

만약 다른 트랜잭션이 데이터를 수정하려고 할 때는 락을 걸어둔 트랜잭션이 끝나기까지 기다리거나 바로 에러를 발생시킬 수 있다.
`START TRANSACTION`으로 트랜잭션을 시작한 뒤 `FOR UPDATE`를 적용하여 락을 걸 수 있다.

## InnoDB의 FOR UPDATE locking

`Record Locks`, `Gap Locks`, `Next-Key Locks`, `Insert Intention Locks`이 존재한다.
각 락에 대한 자세한 설명은 [여기](https://dev.mysql.com/doc/refman/8.4/en/innodb-locking.html)를 참고하자.

### 1) Record Locks

하나의 인덱스 레코드를 잠근다. 
레코드가 잠겼을 때 다른 트랜잭션은 이 레코드의 삽입, 수정, 삭제가 제한된다.

```sql
START TRANSACTION;

SELECT * FROM customer WHERE customer_id = 1 FOR UPDATE;
```

### 2) Gap Locks

REPEATABLE READ 격리 수준에서 사용되는 락으로 **팬텀 읽기(Phantom Read)** 문제를 방지할 때 사용된다.
팬텀 읽기란 하나의 트랜잭션이 데이터를 두 번 읽는 경우 발생한다.
A 트랜잭션이 데이터를 한번 읽고 이후 B 트랜잭션이 범위 내 새로운 데이터를 삽입하여 커밋한다.
이후 A 트랜잭션이 데이터를 읽으면 B 트랜잭션이 추가한 값을 보게 된다.
A 트랜잭션의 입장에서는 갑자기 없던 데이터가 생기는 것이다.

**Gap Locks는 A 트랜잭션이 읽는 범위 내에 다른 트랜잭션이 삽입을 못하도록 막아 팬텀 읽기가 발생하는 문제를 해결한다.**
Gap Locks는 **범위 내** 새로운 삽입만을 막으며 기존 레코드의 수정과 삭제를 막는 것은 Record Locks이 수행한다.

아래의 트랜잭션은 다른 트랜잭션이 10 ~ 20 사이에 새로운 데이터를 추가하는 것을 막아 하나의 트랜잭션 내에서 10 ~ 20 사이 데이터에 대한 SELECT를 여러 번 하더라도 일관된 결과를 얻을 수 있다.

```sql
START TRANSACTION;

SELECT c1 FROM t WHERE c1 BETWEEN 10 and 20 FOR UPDATE;
```

### 3) Next-Key Locks

Record Locks와 Gap Locks를 결합한 락으로 특정 인덱스 레코드 자체를 잠금과 동시에 앞에 있는 레코드 사이의 Gap도 함께 잠그게 된다.
예를 들어 인덱스에 10, 13이 있다고 가정하면 13에 대한 Next-Key Locks는 13 인덱스 레코드를 잠그고(Record Locks), (10, 13] 범위도 함께 잠근다(Gap Locks).

---
# 3. GROUP BY 
`GROUP BY`는 보통 집계 함수와 같이 사용되며 특정 컬럼을 기준으로 최대값(MAX), 건수(COUNT), 합계(SUM), 평균(AVG)과 같은 집계성 데이터를 추출할 때 사용한다.
`GROUP BY` 절 뒤에는 하나 이상의 속성이 올 수 있다.
아래의 SQL 문은 달마다 매출을 확인한다.
예를 들어 2024년 8월의 모든 매출을 합산하여 하나의 튜플이 된다.

```sql
SELECT sales_year, sales_month, SUM(sales)
FROM incomes
GROUP BY sales_year, sales_month
```

## 주의할 점
MySQL에서 sql_mode가 only_full_group_by로 적용된 경우 다음과 같은 제약 사항이 있다.
`GROUP BY`를 사용할 경우 SELECT 할 수 있는 컬럼은
1. `GROUP BY`에 나열된 컬럼
2. 집계 함수
만 사용할 수 있다.

sql_mode가 only_full_group_by인 경우 아래와 같은 쿼리는 에러가 발생한다.
id는 GROUP BY된 속성도 아니고 집계 함수도 아닌 비그룹 속성이다.
DBMS 입장에서는 그룹화할 때 어떤 id를 반환해야 하는지 알 수 없기 때문이다.
```sql
SELECT
  id,
  revenue_month,
  MAX(revenue) AS revenue
FROM Orders
GROUP BY revenue_month
```

---
# 4. ORDER BY
SELECT하여 나온 결과를 정렬할 때 사용하는 함수이다.

## MySQL ORDER BY 최적화
만약 정렬하고자 하는 속성에 인덱스가 걸려 있다면 기존 인덱스를 활용하여 불필요한 정렬 과정을 생략한다.
하지만 인덱스가 없다면 Filesort 방식을 활용하여 데이터를 정렬한다.

### 1) 인덱스를 활용한 정렬
MySQL의 인덱스는 이미 정렬된 데이터 구조로 이를 활용하면 추가적인 정렬 작업 없이도 데이터를 정렬할 수 있다.
따라서 Filesort를 활용한 정렬보다 일반적으로 빠르다.
하지만 `ORDER BY`에서 인덱스 속성이 사용되더라도 계산식이나 집계 함수 등이 사용되면 인덱스를 사용할 수 없다.
또한 정렬에 사용되는 키가 첫 번째 인덱스부터 연속되는 **prefix**여야만 인덱스를 활용하여 정렬할 수 있다.
인덱스가 (key_part1, key_part2)일 때 인덱스 정렬이 가능한 경우는 다음과 같다.

```sql
-- 모든 속성을 가져오는 경우 인덱스를 사용해 정렬한 후 다른 속성을 직접 조회해야 한다.
-- 이 경우 조회하는 비용이 인덱스를 활용한 이득보다 크다고 판단되면 옵티마이저는 Filesort를 사용할 수 있다.
SELECT * FROM t1 ORDER BY key_part1, key_part2;

-- 첫번째 인덱스 속성을 상수로 고정하므로 다음 인덱스를 사용할 수 있다.
SELECT * FROM t1 WHERE key_part1 = constant ORDER BY key_part2;
  
-- 첫 번째 인덱스 만으로 인덱스를 활용하여 정렬할 수 있다. 
SELECT * FROM t1 ORDER BY key_part1;
```

인덱스를 활용할 수 없는 경우는 다음과 같다.

```sql
-- 첫 번째 인덱스를 건너뛰고 정렬할 수는 없다. prefix가 아니다.
SELECT * FROM t1 ORDER BY key_part2;
  
-- 집계함수가 포함되어 있으므로 인덱스 정렬할 수 없다.
SELECT * FROM t1 ORDER BY ABS(key_part1);
```

### 2) Filesort 정렬
인덱스를 활용할 수 없을 때 Filesort 정렬을 사용한다.
SELECT 결과가 크지 않다면 메모리에서 정렬 작업을 완료하고 그렇지 않다면 MySQL은 임시 파일을 생성하여 정렬 작업을 수행한다.
임시 파일을 사용하는 방식은 I/O 등의 작업으로 메모리의 작업보다 더 많은 시간이 소요된다.

---
# Reference

### 1
[https://towardsdatascience.com/the-6-steps-of-a-sql-select-statement-process-b3696a49a642](https://towardsdatascience.com/the-6-steps-of-a-sql-select-statement-process-b3696a49a642)

### 2
[https://miintto.github.io/docs/mysql-select-for-update](https://miintto.github.io/docs/mysql-select-for-update)

[https://dev.mysql.com/doc/refman/8.4/en/innodb-locking.html](https://dev.mysql.com/doc/refman/8.4/en/innodb-locking.html)

### 3
[https://velog.io/@devjooj/MySQL-EP-1.-GROUP-BY](https://velog.io/@devjooj/MySQL-EP-1.-GROUP-BY)

[https://school.programmers.co.kr/questions/38703](https://school.programmers.co.kr/questions/38703)

### 4
[https://dev.mysql.com/doc/refman/8.4/en/order-by-optimization.html](https://dev.mysql.com/doc/refman/8.4/en/order-by-optimization.html)
