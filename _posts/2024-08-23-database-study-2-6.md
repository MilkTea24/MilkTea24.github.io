---
title: 데이터베이스 스터디 2주차 - 서브 쿼리, DISTINCT, 페이징
author: milktea
date: 2024-08-23 10:00:00 +0800
categories: [Database]
tags: [SQL]
pin: true
math: true
mermaid: true
---
# 1. 서브 쿼리(SubQuery)
서브쿼리란 다른 쿼리 내부에 포함되어 있는 SELECT 문을 의미한다.
다른 쿼리의 내부에 포함되어 있어 메인 쿼리의 조건을 결정한다.

## 서브 쿼리 실행 과정
**서브 쿼리 실행 -> 메인 쿼리 실행**
이 때 서브 쿼리는 메인 쿼리의 속성을 사용할 수 있지만 메인 쿼리는 서브 쿼리의 속성을 사용할 수 없다.

## 서브 쿼리 종류
### 1) WHERE 문
다음과 같이 서브 쿼리를 활용할 수 있다.

```sql
# 주문 테이블에서 주문금액이 평균 이상만 출력
SELECT * FROM orders WHERE total_amount > (SELECT AVG(total_amount) FROM orders);

# 2번 구매자가 주문한 금액들 중 하나라도 일치하는 금액을 가진 튜플들을 반환
SELECT * FROM orders WHERE total_amount = ANY (SELECT total_amount FROM orders WHERE customer_id = 2);

# 2번 구매자가 주문한 모든 금액들보다 큰 주문 금액인 튜플들만 반환
SELECT * FROM orders WHERE total_amount > ALL (SELECT total_amount FROM orders WHERE customer_id = 2);
```

### 2) FROM 문
서브 쿼리가 FROM 절에 사용된 경우 무조건 **AS 별칭**을 지정해 주어야 한다.

```sql
SELECT sub_orders.total_amount FROM (SELECT * FROM orders WHERE customer_id = '1') sub_orders;
```

### 3) 스칼라 서브 쿼리
SELECT 문에서도 서브 쿼리가 올 수 있다.
오직 **하나의 레코드만 리턴**할 수 있다.

```sql
# 조인 없이 서브 쿼리를 사용하여 username을 출력
SELECT orders.total_amount, (SELECT username FROM customers WHERE orders.customer_id = customers.customer_id) FROM orders; 
```

이 예시를 포함하여 서브 쿼리는 다음과 같은 구문 안에 들어갈 수 있다.
1. WHERE
2. FROM
3. SELECT
4. HAVING
5. ORDER BY
6. INSERT문의 VALUES
7. UPDATE문의 SET

## 적절한 서브 쿼리 사용 방법
서브 쿼리는 위와 같이 유연하게 데이터를 가지고 올 수 있는 장점이 있지만 최적화가 불가능하고 매번 SELECT 문이 실행되는 등 성능이 떨어지는 문제가 있을 수 있다.
따라서 서브 쿼리의 결과 값이 아주 크거나 쿼리가 복잡한 경우 조인을 활용하는 등 효율적인 쿼리 개선이 필요하다.

하지만 단순한 결과를 반환하거나 집계, 계산등의 함수를 활용해야 할 일이 있을 때 가독성, 단순성 면에서는 서브 쿼리를 활용하는 방법이 더 좋을 수도 있다.

---
# 2. DISTINCT

`SELECT`한 결과에서 중복을 제거할 때 사용하는 키워드이다.
`SELECT` 구문이 실행되고 `ORDER BY`를 수행하기 이전에 중복을 제거한다.

## DISTINCT 예시

| id | state | city          | street_name         |
|----|-------|---------------|---------------------|
| 1  | NY    | New York City | Broadway            |
| 2  | CA    | Los Angeles   | Sunset Boulevard    |
| 3  | IL    | Chicago       | Michigan Avenue     |
| 4  | TX    | Houston       | Westheimer Road     |
| 5  | CA    | Los Angeles   | Hollywood Boulevard |
| 6  | NY    | New York City | Fifth Avenue        |
| 7  | CA    | San Francisco | Lombard Street      |

### 1)

```sql
SELECT DISTINCT state FROM address;
```

state에서 중복된 값을 제거하여 반환한다.

| state |
|-------|
| NY    | 
| CA    |
| IL    |
| TX    |

### 2)

```sql
SELECT DISTINCT state, city FROM address;
```

state와 city의 조합 중 중복되는 값을 제거하여 반환한다.

| state | city          |
|-------|---------------|
| NY    | New York City |
| CA    | Los Angeles   |
| IL    | Chicago       |
| TX    | Houston       |
| CA    | San Francisco |

---
3. 페이지네이션(Pagination)
페이지 번호, 페이지당 출력할 데이터의 수 등으로 수많은 데이터 목록의 일부만 가져와서 보여주는 UI 요소를 페이지네이션이라고 한다.
이를 위해 DB에서 일부만 가져오는 과정을 페이징(Paging)이라고 한다.

## Standard SQL 구현
```sql
SELECT * FROM orders ORDER BY total_amount DESC LIMIT 10, 5; 

SELECT * FROM orders ORDER BY total_amount DESC LIMIT 5 OFFSET 10;
```

두 쿼리 모두 처음 10개 행을 건너뛰고(OFFSET) 11번째 ~ 15번째 데이터만 가져오는 코드이다.

## Spring Data JPA
이전 프로젝트에서 Pagenation을 활용한 사례를 예시로 가져왔다.
요구 사항은 다음과 같다.
1. 식당의 리뷰를 최신 순, 평점 높은 순, 추천 지수 높은 순으로 정렬할 수 있어야 한다.
2. 리뷰는 페이지 당 5개씩 가져온다.

### 1) ReviewController에서 Pageable 객체 생성
```java
PageRequest.of(Integer.parseInt(page),
                Integer.parseInt(size),
                Sort.by(Sort.Direction.DESC, sort));
```

`PageRequest.of`에서 첫 번째 매개 변수를 페이지 번호, 두 번째를 페이지 당 개수, 세 번째를 정렬할 속성으로 하여 Pageable 객체를 생성한다.
PageRequest는 Spring Data JPA에서 제공하는 Pageable 구현체 중 하나로 페이지 정보를 생성하는 클래스이다.

### 2) ReviewService에서 ReviewRepository 호출
```java
public Page<GetReviewsPagesResponse> getReviewsByPlaceId(Long id, Pageable pageable) {
    return reviewRepository.findAllByPlaceId(id, pageable).map(Review::mapToGetReviewsPagesResponse);
}
```
ReviewRepository의 메서드 매개변수에 Pageable 객체를 함께 전달하면 결과를 페이징해서 반환한다.


### 3) ReviewRepository
```java
Page<Review> findAllByUserId(Long userId, Pageable pageable);
```
메서드 정의는 이와 같이 Pageable을 매개 변수를 전달하고 리턴 값은 Page 객체로 하면 된다.
Page 객체는 반환할 데이터의 정보와 함께 페이징 정보를 제공하는 인터페이스로써 페이징한 데이터와 함께 전체 페이지 수, 현재 페이지 수 등을 알 수 있다.

### 4) ReviewController에서 출력
```java
  Page<GetReviewsPagesResponse> reviewResponse = reviewService.getReviewsByPlaceId(id, pageableWrapper.pageable());
  for (GetReviewsPagesResponse response : reviewResponse) {
  ...
  }
  //현재 페이지 번호
  System.out.println(reviewResponse.getPageable().getPageNumber());
  //총 페이지 번호
  System.out.println(reviewResponse.getTotalPages());
```
위와 같이 Page 객체는 Iterable을 구현하므로 Page 객체 데이터를 순회하여 리뷰를 출력할 수 있으며 현재 페이지 번호, 총 페이지 번호도 함께 얻을 수 있다.

---
# Reference

### 1
[https://inpa.tistory.com/entry/MYSQL-%F0%9F%93%9A-%EC%84%9C%EB%B8%8C%EC%BF%BC%EB%A6%AC-%EC%A0%95%EB%A6%AC](https://inpa.tistory.com/entry/MYSQL-%F0%9F%93%9A-%EC%84%9C%EB%B8%8C%EC%BF%BC%EB%A6%AC-%EC%A0%95%EB%A6%AC)

[https://12bme.tistory.com/299](https://12bme.tistory.com/299)

### 3
[https://velog.io/@kimdy0915/JPA-Spring-Data-Jpa-Pageable%EB%A1%9C-Pagination-%EC%89%BD%EA%B2%8C-%EA%B5%AC%ED%98%84%ED%95%98%EA%B8%B0](https://velog.io/@kimdy0915/JPA-Spring-Data-Jpa-Pageable%EB%A1%9C-Pagination-%EC%89%BD%EA%B2%8C-%EA%B5%AC%ED%98%84%ED%95%98%EA%B8%B0)
