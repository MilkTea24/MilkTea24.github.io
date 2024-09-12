---
title: DB - 조인
author: milktea
date: 2024-08-22 12:00:00 +0800
categories: [Database]
tags: [SQL]
pin: true
math: true
mermaid: true
---

> 데이터베이스 스터디 2주차에서 학습하고 정리한 내용입니다.
{: .prompt-info }

# 1. Join
관계 대수에서 조인은 Cross Product한 결과에 여러 조건을 추가한 것이다.
Cross Product한 결과에 조건에 맞는 결과만 Selection하거나 Projection 한다.
![img.png](/assets/img/posts/database/study-2-5/img.png)

## 예제 테이블

### customers

| customer_id | customer_name |
|-------------|---------------|
| 1           | Kim           |
| 2           | Lee           |
| 3           | Park          |

### orders

| order_id | customer_id | total_amount |
|----------|-------------|--------------|
| 1        | 1           | 8000         |
| 2        | 1           | 3000         |
| 3        | 2           | 8000         |
| 4        | 2           | 9000         |
| 5        | 4           | 11000        |

---
# 2. Cross Join
Cross Join은 관계 대수에서 Cross Product를 적용한 결과와 같다.
Cross Product는 두 테이블의 튜플을 조합하여 가능한 모든 경우의 수이다.
Cross Join의 튜플 수는 각 테이블의 튜플 수를 곱한 값과 일치한다.

---
# 3. Inner Join
Cross Join에서 조인 조건과 일치하는 결과만 반환하면 **Inner Join**이다.
조인 조건과 일치하는 튜플들만 남고 나머지는 다 버려진다.

## 1) Theta Join(θ)
Cross Product한 결과의 조건으로 =, >, <, >=, <= 등으로 표현되면 **Theta Join**이라고 한다.

```sql
SELECT * FROM customers CROSS JOIN orders 
ON customers.customer_id > orders.customer_id;
```

| customer_id | customer_name | order_id | customer_id(orders) | total_amount |
|-------------|---------------|----------|---------------------|--------------|
| 2           | Lee           | 1        | 1                   | 8000         |
| 2           | Lee           | 2        | 1                   | 3000         |
| 3           | Park          | 1        | 1                   | 8000         |
| 3           | Park          | 2        | 1                   | 3000         |
| 3           | Park          | 3        | 2                   | 8000         |
| 3           | Park          | 4        | 2                   | 9000         |

## 2) Equi Join
세타 조인 중 조건 θ가 =인 특별한 경우를 **Equi Join**이라고 한다.
MySQL에서 `JOIN`은 Equi Join을 의미한다.

```sql
SELECT * FROM customers CROSS JOIN orders 
ON customers.customer_id = orders.customer_id;
```

| customer_id | customer_name | order_id | customer_id(orders) | total_amount |
|-------------|---------------|----------|---------------------|--------------|
| 1           | Kim           | 1        | 1                   | 8000         |
| 1           | Kim           | 2        | 1                   | 3000         |
| 2           | Lee           | 3        | 2                   | 8000         |
| 2           | Lee           | 4        | 2                   | 9000         |

## 3) Natural Join
Equi Join은 동일한 이름을 가진 customer_id 열이 중복해서 나오고 있다.
이 Equi Join에서 중복되어 나오는 customer_id 열을 하나로 통합하면 자연 조인이 된다.
```sql
SELECT * FROM customers NATURAL JOIN orders;
```


| customer_id | customer_name | order_id | total_amount |
|-------------|---------------|----------|--------------|
| 1           | Kim           | 1        | 8000         |
| 1           | Kim           | 2        | 3000         |
| 2           | Lee           | 3        | 8000         |
| 2           | Lee           | 4        | 9000         |

---
# 4. Outer Join
**Inner Join**과 달리 조인 조건을 만족하지 않는 데이터도 포함될 수 있다.

## 1) Left Outer Join
조인 조건을 만족하는 데이터에 조인 조건을 만족하지 않는 왼쪽 테이블의 데이터도 모두 추가한다.
customers.customer_id = orders.customer_id 조인 조건을 만족하지 않아 포함되지 않았던 Park이 포함되어 있다.
order table의 정보는 null로 표시되어 있다.

| customer_id | customer_name | order_id | customer_id(orders) | total_amount |
|-------------|---------------|----------|---------------------|--------------|
| 1           | Kim           | 1        | 1                   | 8000         |
| 1           | Kim           | 2        | 1                   | 3000         |
| 2           | Lee           | 3        | 2                   | 8000         |
| 2           | Lee           | 4        | 2                   | 9000         |
| 3           | Park          | null     | null                | null         |

## 2) Right Outer Join
조인 조건을 만족하는 데이터에 조인 조건을 만족하지 않는 오른쪽 테이블의 데이터도 모두 추가한다.
조인 조건을 만족하지 않아 포함되지 않았던 5번째 주문 내역이 포함되어 있다.

| customer_id | customer_name | order_id | customer_id(orders) | total_amount |
|-------------|---------------|----------|---------------------|--------------|
| 1           | Kim           | 1        | 1                   | 8000         |
| 1           | Kim           | 2        | 1                   | 3000         |
| 2           | Lee           | 3        | 2                   | 8000         |
| 2           | Lee           | 4        | 2                   | 9000         |
| null        | null          | 5        | 4                   | 11000        |

## 3) Full Outer Join
조인 조건을 만족하는 데이터에 조인 조건을 만족하지 않는 왼쪽, 오른쪽 테이블의 데이터를 모두 추가한다.

| customer_id | customer_name | order_id | customer_id(orders) | total_amount |
|-------------|---------------|----------|---------------------|--------------|
| 1           | Kim           | 1        | 1                   | 8000         |
| 1           | Kim           | 2        | 1                   | 3000         |
| 2           | Lee           | 3        | 2                   | 8000         |
| 2           | Lee           | 4        | 2                   | 9000         |
| 3           | Park          | null     | null                | null         |
| null        | null          | 5        | 4                   | 11000        |

---
# Reference

### 0
그림 출처: [https://medium.com/@iammanolov98/mastering-sql-joins-coding-interview-preparation-cross-join-230abb340336](https://medium.com/@iammanolov98/mastering-sql-joins-coding-interview-preparation-cross-join-230abb340336)

### 1
홍봉희 편저, 데이터베이스 SQL 프로그래밍 'MySQL 실습', 부산대학교출판문화원

### 2
[https://velog.io/@haerong22/LEFT-OUTER-JOIN-%EC%9D%98-%ED%95%A8%EC%A0%95](https://velog.io/@haerong22/LEFT-OUTER-JOIN-%EC%9D%98-%ED%95%A8%EC%A0%95)
