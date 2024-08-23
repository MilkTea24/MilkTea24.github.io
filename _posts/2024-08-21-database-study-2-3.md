---
title: 데이터베이스 스터디 2주차 - 뷰
author: milktea
date: 2024-08-21 10:00:00 +0800
categories: [Database]
tags: [View, Integrity]
pin: true
math: true
mermaid: true
---

# 1. 뷰(View)
뷰를 사용하는 가장 큰 이유는 **논리적 데이터 독립성**을 지원하기 때문이다.
개념 스키마의 테이블인 기본 테이블이 변경되어도 외부 스키마에서 사용하는 뷰가 변경되지 않으면 응용 프로그램은 영향을 받지 않는다.
예를 들어, 개념 스키마 테이블에 새로운 컬럼이 추가, 테이블이 두 개 이상의 테이블로 분할되거나 합쳐지는 경우가 있더라도 응용 프로그램이 보는 데이터는 변화가 없다.

## 장점
1. 논리적 데이터 독립성 지원
2. 응용 프로그램, 사용자에게 필요한 정보만 노출시킬 수 있다. 보안 면에서 유리하다.
3. 복잡한 조인 처리를 뷰로 미리 정의할 수 있어 검색 관리성이 크게 증가한다.

## 뷰 테이블의 생성
```sql
CREATE VIEW user_orders AS
SELECT 
    users.id AS user_id, 
    users.name AS username,
    orders.id AS order_id,
    orders.order_price
FROM 
    users
INNER JOIN
    orders ON users.id = orders.user_id;
```

이 SQL 쿼리와 같이 조인한 결과를 뷰로 생성할 수도 있다.

## 뷰 테이블의 수정
뷰 테이블은 실제 데이터를 가지고 있지 않으므로 개념 스키마의 테이블이 갱신되어야 한다.
이 때 항상 뷰를 통해 데이터를 수정할 수 있는 것은 아니다.
어디까지나 뷰 테이블은 가상 테이블이므로 뷰 테이블을 통해 수정할 때는 몇가지 주의해야 할 사항이 있다.

### With Check Option
`WITH CHECK OPTION`은 뷰에서 허용된 데이터만 삽입, 업데이트할 수 있도록 제한한다.
다음과 같은 뷰를 생성했다고 하자.

```sql
CREATE VIEW vip_users_view AS
SELECT id, name, status
FROM users
WHERE status = 'VIP'
WITH CHECK OPTION;
```

이 때 `WITH CHECK OPTION`이 없을 경우 vip_users_view를 통해 VIP가 아닌 유저 또한 삽입 및 삭제가 가능하다.
이러한 작업은 데이터 일관성에 좋지 않은 영향을 미치므로 `WITH CHECK OPTION`을 통해 뷰의 조건을 만족하는 데이터만 삽입 및 삭제할 수 있도록 강제할 수 있다.

### 뷰를 수정할 수 있는 경우
뷰가 조인으로 생성되었거나 집계 함수, distinct, group by, having, union, subquery으로 정의되어 **뷰와 테이블의 데이터가 1대1 대응 관계가 되지 않는 경우** 수정이 불가능하다.
뷰를 생성한 경우 뷰의 테이블과 기본 테이블의 데이터가 1대1 대응 관계일 때만 가능하다.

---
# Reference
홍봉희 편저, 데이터베이스 SQL 프로그래밍 'MySQL 실습', 부산대학교출판문화원


