---
title: 데이터베이스 스터디 3주차 - 쿼리 실행 계획과 힌트

author: milktea
date: 2024-08-29 08:00:00 +0800
categories: [Database]
tags: [ExecutionPlan]
pin: true
math: true
mermaid: true
---
# 1. 쿼리 실행 계획

현재 대부분의 옵티마이저는 비용 기반 옵티마이저로 여러 개의 옵션에 대한 비용을 예측한다.
이를 예측하여 쿼리 실행 예상 정보를 정리한 내용이 바로 쿼리 실행 계획이다.
항상 옵티마이저가 이상적인 판단을 할 수는 없기 때문에 DBMS 서버는 문제점을 파악할 수 있도록 옵티마이저가 수립한 실행 계획을 확인할 수 있게 해준다.

## 쿼리 실행 계획 생성 및 실행 과정

1. SQL 문법 해석 및 분석
2. 최적화(여러 정보를 바탕을 최적 계획 선택)
3. 실행 계획 수립(선택된 계획으로 실제 데이터에 접근)
4. 실행

## MySQL 실행 계획 구성 요소
###  통계 정보

비용 기반 최적화에서 가장 중요한 것은 통계 정보다.
이 통계 정보가 정확해야 정확한 비용을 계산할 수 있다.

현재 MySQL은 통계 정보의 정확성을 높이기 위해 통계 정보를 영속화하고 통계 정보를 수집할 수 있는 여러 옵션들을 제공하고 있다.

### 히스토그램
5.7 이전에는 인덱스된 컬럼의 유니크한 값의 개수 정도만 가지고 실제 분포는 알지 못했다.
하지만 8.0 버전으로 업그레이드 되면서 컬럼의 데이터 분포를 참조할 수 있는 히스토그램 정보를 활용할 수 있다.

```mysql
-- 히스토그램 분석
ANALYZE TABLE employees UPDATE HISTOGRAM ON gender, hire_date;

-- 히스토그램 정보 조회
SELECT *
FROM COLUMN_STATISTICS
WHERE SCHEMA_NAME='employees'
AND TABLE_NAME='employees';
```

이 히스토그램을 사용한 실행 계획은 더 정확하게 비용을 예측할 수 있다.

## 실행 계획 확인
`EXPLAIN` 키워드를 사용하여 실행 계획을 확인할 수 있다.
```mysql
EXPLAIN
SELECT *
FROM employees e
    INNER JOIN salaries s ON s.emp_no = e.emp_no
WHERE first_name='ABC';
```

### 쿼리의 실행 시간 확인 
`EXPLAIN ANALYZE` 키워드를 통해 쿼리의 실행 계획과 단계별 소요된 시간 정보를 확인할 수 있다.

---
# 2. 힌트

옵티마이저가 많이 발전해도 옵티마이저가 쿼리 실행 계획을 비즈니스 모델에 맞게 100% 완벽하게 작성해주지는 못한다.
이 때 옵티마이저에게 쿼리 실행 계획을 어떻게 수립해야 할지 개발자나 DBA가 알려줄 수 있는데 일반적인 RDBMS는 이 목적을 위해 힌트가 제공된다.
MySQL에서는 인덱스 힌트와 옵티마이저 힌트를 제공한다.

## 인덱스 힌트

옵티마이저 힌트가 도입되기 전 ANSI-SQL 표준 문법을 준수하지 않는 힌트들이다.
SELECT 명령과 UPDATE 명령에서만 사용할 수 있다.

### STRAIGHT_JOIN

`STRAIGHT_JOIN`은 옵티마이저가 조인 순서를 변경하지 못하도록 고정하는 역할을 한다.
A와 B 테이블을 조인할 때 A 테이블은 조건을 만족하는 레코드 수가 5000만 건이고 B 테이블은 조건을 만족하는 레코드 수가 1000건이라고 하자.
A 테이블이 먼저 조회된다면 5000만 번을 반복하여 B 테이블을 탐색하고 반대라면 1000번 A 테이블을 탐색한다.

따라서 조인 순서는 성능에 많은 영향을 미칠 수 있는데 옵티마이저는 소요되는 예상 비용을 바탕으로 조인 순서를 결정한다.
실행 계획을 확인하여 옵티마이저가 조인 순서를 잘못 수립하였다면 STRAIGHT_JOIN 힌트를 이용해 순서를 지정하는 것이 좋다.

### USE INDEX

실행 계획을 확인하여 옵티마이저가 인덱스를 잘못 선택했다면 `USE INDEX` 힌트를 사용하여 강제로 특정 인덱스를 사용하도록 만들 수 있다.
`IGNORE INDEX`는 강제로 특정 인덱스를 사용하지 못하게 한다.
사용자가 터무니 없는 인덱스 힌트를 지정했다면 MySQL은 해당 힌트를 무시한다.

```mysql
SELECT * FROM employees USE INDEX(primary) WHERE emp_no=10001;

SELECT * FROM employees IGNORE INDEX(primary) WHERE emp_no=10001;
```

## 옵티마이저 힌트
MySQL 8.0 기준으로 아주 다양한 힌트가 있으며 영향 범위에 따라 크게 4가지 그룹으로 나눌 수 있다.
- 인덱스: 특정 인덱스의 이름을 사용할 수 있는 옵티마이저 힌트
- 테이블: 특정 테이블의 이름을 사용할 수 있는 옵티마이저 힌트
- 쿼리 블록: 특정 쿼리 블록에 사용할 수 있는 옵티마이즈 힌트
- 글로벌: 전체 쿼리에 영향을 미치는 힌트

### MAX_EXECUTION_TIME

쿼리의 최대 실행 시간을 밀리초 단위로 설정하여 지정된 시간을 초과하면 쿼리를 실패하게 만들 수 있다.

```mysql
SELECT /*+ MAX_EXECUTION_TIME(500) */ *
FROM employees
ORDER BY last_name LIMIT 1;
```

### SET_VAR

SET_VAR 힌트는 일시적으로 시스템 변수를 변경하여 쿼리의 실행 계획을 변경할 수도 있다.
예를 들어 대용량 처리 쿼리일 때 버퍼의 크기를 일시적을 증가시켜 처리 속도를 향상할 수 있다.

```mysql
SELECT /*+ SET_VAR(join_buffer_size = 8388608) */ *
FROM employees e
JOIN departments d ON e.dept_no = d.dept_no
WHERE e.hire_date > '2000-01-01';
```

### JOIN_FIXED_ORDER, JOIN_ORDER, JOIN_PREFIX, JOIN_SUFFIX

기존의 STRAIGHT_JOIN 힌트는 일부 조인 순서를 강제할 수는 없었다.
또한 조인 순서를 변경할 때 쿼리를 직접 변경해야 하는 번거로움이 있었다.
이 문제를 보완하기 위해 4가지 옵티마이저 힌트를 제공한다.

- JOIN_FIXED_ORDER: STRAIGHT_JOIN과 동일한 기능
- JOIN_ORDER(테이블명, 테이블명, ...): 힌트에 명시된 테이블 순서대로 조인을 실행
- JOIN_PREFIX: 드라이빙 테이블만 강제
- JOIN_SUFFIX: 드리븐 테이블(가장 마지막에 조인되어야 할 테이블)만 강제

### SKIP_SCAN, NO_SKIP_SCAN

MySQL 옵티마이저가 유니크한 값의 개수를 제대로 분석하지 못해 비효율적인 인덱스 스킵 스캔을 선택하면 `NO_SKIP_SCAN` 옵티마이저 힌트를 이용하여 인덱스 스킵 스캔을 사용하지 않게 할 수 있다.

```mysql
SELECT /*+ NO_SKIP_SCAN(employees ix_gender_birthdate) */ gender, birth_date
FROM employees
WHERE birth_date >= '1965-02-01';
```

---
# Reference
백은빈, 이성욱 편저, Real MySQL 8.0, 위키북스

### 1
[https://seung.tistory.com/entry/MySQL-%EC%8B%A4%ED%96%89-%EA%B3%84%ED%9A%8DQuery-Plan%EC%9D%B4-%EB%AD%94%EB%8D%B0](https://seung.tistory.com/entry/MySQL-%EC%8B%A4%ED%96%89-%EA%B3%84%ED%9A%8DQuery-Plan%EC%9D%B4-%EB%AD%94%EB%8D%B0)

### 2
[https://devuna.tistory.com/36](https://devuna.tistory.com/36)

