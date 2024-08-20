---
title: 데이터베이스 스터디 2주차 - SQL의 정의 및 특성
author: milktea
date: 2024-08-15 10:00:00 +0800
categories: [Database]
tags: [Schema, DBMS]
pin: true
math: true
mermaid: true
---

# 1. SQL이란?
SQL은 구조화 질의어(Structured Query Language)의 약자로 데이터베이스의 데이터를 조작하고 검색할 수 있는 언어이다.
관계 대수와 관계 해석 이 두 가지 수학적 쿼리 언어가 이 SQL의 바탕이 된다.
관계 대수와 관계 해석의 장점을 적절히 가져오면서 추가적인 기능을 지원하여 현재의 SQL이 되었다.

## 관계 대수(Relational Algebra)
관계 대수는 절차적인 특성을 가진다.
관계 대수는 데이터를 '어떻게' 가져오는 지에 대한 방법을 기술하고 있으며 다음과 같은 연산이 존재한다.

| 연산자 명          | 연산자      | 설명                                                                           |
|----------------|----------|------------------------------------------------------------------------------|
| Selection      | σ        | 릴레이션에서 일부 튜플을 가져온다.                                                          |
| Projection     | π        | Projection 리스트에 없는 속성을 삭제한다(=원하는 속성만 확인한다).                                  |
| Union          | ∪        | 두 릴레이션의 튜플을 합하며 이때 중복된 튜플은 제거                                                |
| Cross-Product  | ×        | 두 릴레이션의 튜플들로 조합할 수 있는 모든 경우를 출력한다.<br/> 이 결과로 생성된 튜플의 수는 두 릴레이션의 튜플 곱과 일치한다. |
| Set-difference | −        | A-B에서 A에는 존재하지만 B에는 존재하지 않는 튜플을 출력한다.                                        |
| Join           | ⨝(세타 조인) | Cross-Product에 여러 제약 사항을 추가하여 구현한다.                                          |
| Division       | /        | A/B에서 모든 B의 튜플 y에 대해 xy 튜플이 A에 존재하는 A의 튜플 x들을 반환                             |

하지만 관계 대수는 정렬이 안되고 산술 연산을 할 수 없는 단점을 가지고 있다.
또한 데이터 베이스를 수정할 수 없다.
SQL은 관계 대수의 개념을 가져오면서 이러한 관계 대수의 문제를 해결하였다.

## 관계 해석(Relational Calculus)
관계 해석은 관계 대수와 달리 비절차적이다.
또한 사용자가 어떻게 검색하는지에 대한 정의 없이 논리적 조건을 제시하면 결과를 반환해주는 선언적인 특성을 가진다.
이 Relational Calculus에서 Tuple을 값으로 가지면 Tuple Calculus가 된다. 
`{t| t ∈ loan  ∧ t[amount]>=10000}` 이와 같이 찾고자 하는 Tuple의 논리적 조건(loan 릴레이션에서 값이 10000 이상인 튜플)만 명시하면 소프트웨어가 이와 일치하는 결과를 반환한다.

## SQL이 다른 프로그래밍 언어와 다른 점은?
표준 SQL은 C와 Java와 같은 다른 프로그래밍 언어와 다르게 **튜링 완전하지 않다**.
SQL은 if-else와 같은 조건 분기, for와 같은 loop 기능이 제한되어 있다.
또한 SQL은 메모리를 임의로 할당하고 사용할 수 없다.

튜링 완전한 언어가 되기 위해서는 메모리를 조작할 수 있고 조건 분기, Loop 등을 지원하여 모든 연산을 수행할 수 있어야 한다.
Java와 C는 이를 수행할 수 있기 때문에 튜링 완전 언어이고 SQL은 이를 수행할 수 없기 때문에 튜링 완전하지 않은 언어이다.

SQL이 튜링 완전하지 않은 방식을 택함으로써 얻는 장점도 있다.
무한 루프가 없으므로 안정성 면에서 유리하며 선언적 특성을 채택하여 간결하고 직관적으로 사용할 수 있다.

# 2. SQL 실행 과정
세부 구현은 다를 수 있지만 많은 RDBMS는 MySQL과 비슷한 SQL 처리 과정을 거친다.
MySQL SQL 처리 과정은 다음과 같다.

## MySQL SQL 처리 과정
![img.png](/assets/img/posts/database/study-2-1/img.png)

### 1. Parsing
  Client가 작성한 SQL문이 MySQL에서 사용할 수 있는 문법을 따르는지 **Parser**가 확인한다.
  이후 입력된 쿼리를 컴퓨터가 처리할 수 있도록 Parse Tree를 만든다.

### 2. PreProcessor
  문법에 이상이 없다면 추가적인 검사를 수행한다,
  SQL을 테이블에서 수행할 수 있는지 확인하고 또한 SQL 문을 수행할 수 있는 권한이 Client에 있는지 확인한다.

### 3. Query Optimization

  **Query Optimizer**는 Parse Tree를 어떻게 실행할 지 최적의 옵션을 찾는다.
  Optimizer 전략에는 CBO(Cost Based Optimizer), RBO(Rule Based Optimizer) 등이 있는데 MySQL은 CBO를 기본으로 한다.
  여러 후보 계획에 대한 비용을 계산한 후 최적의 비용을 가지는 계획을 선택한다. 

### 4. Query Execution

  최적화된 실행 계획을 바탕으로 실제로 쿼리를 실행한다.
  **Query Execution Engine**이 이 역할을 수행하며 이 과정에서 실제 데이터를 보관하는 스토리지 엔진과 지속적으로 데이터를 교환한다.

### 5. Storage Engines

  Handler API를 통해 Query Execution Engine과 통신하며 Query Execution Engine의 요청에 따라 데이터를 읽거나 쓴다.

### 6. 결과 반환

  MySQL Server는 Query의 결과를 생성하고 Client에 데이터를 전달한다.

# 3. DML
데이터 조작어(Data Manipulation Language)의 약자로 데이터베이스에 저장된 데이터를 조회, 삽입, 수정, 삭제하는데 사용되는 SQL 명령어 집합이다.
`SELECT`는 조회, `INSERT`는 삽입, `UPDATE`는 수정, `DELETE`는 테이블의 데이터를 삭제할 때 사용된다.
```sql
INSERT INTO Student (sid, name, total_grade) VALUES
('1', 'Kim', 3.2),
('2', 'Lee', 3.8);

SELECT * FROM Student WHERE total_grade > 3.5 ORDER BY total_grade DESC;

UPDATE Student SET total_grade = 4.0 WHERE name = 'Kim';

DELETE FROM Student WHERE name = 'Lee';
```

# 4. DDL
데이터 정의어(Data Definition Language)의 약자로 테이블, 인덱스 등 데이터베이스 객체의 구조를 정의하고 관리하는 명령어 집합이다.
`CREATE`는 데이터베이스 객체를 생성, `SHOW`는 데이터베이스 객체의 정보 확인, `ALTER`는 데이터베이스 객체의 수정, `DROP`/`TRUNCATE`는 데이터베이스 객체를 삭제할 때 사용된다.
`DROP`은 객체 내부의 데이터와 구조에 대한 정보까지 완전히 삭제하지만 `TRUNCATE`는 데이터만 삭제하고 객체의 구조에 대한 정보는 유지된다.
```sql
CREATE TABLE Student (
    sid INT PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    total_grade DOUBLE DEFAULT 0
)

SHOW TABLE STATUS LIKE 'Student';

ALTER TABLE Student ADD COLUMN old TINYINT UNSIGNED NOT NULL;

DROP TABLE STUDENT;

TRUNCATE TABLE STUDENT;
```

# 5. DCL
데이터 제어어(Data Control Language)의 약자로 데이터의 접근과 사용을 제어하는데 사용되는 명령어 집합이다.
주로 데이터베이스 시스템의 보안과 관련된 작업을 수행하며 사용자에게 권한을 부여하거나 취소하는 데 사용한다.
`GRANT`는 데이터베이스 객체에 대한 특정 권한을 사용자에게 부여할 때, `REVOKE`는 권한을 취소할 때 사용한다.

```sql
GRANT ALL PRIVILEGES ON Student To 'admin'@'host';
          
FLUSH PRIVILEGES;
      
REVOKE ALL PRIVILEGES FROM Student To 'admin'@'host';
```

# 6. TCL
트랜잭션 제어어(Transaction Control Language)의 약자로 트랜잭션의 시작, 종료, 제어를 위해 사용되는 명령어 집합이다.
`START TRANSACTION`으로 트랜잭션을 시작할 수 있고 `COMMIT`은 트랜잭션을 커밋할 때, `ROLLBACK`은 트랜잭션을 취소할 때, `SAVEPOINT`는 트랜잭션 내 특정 지점을 저장할 때 사용한다.

# Reference
홍봉희 편저, 데이터베이스 SQL 프로그래밍 'MySQL 실습', 부산대학교출판문화원

### 1
[https://www.cs.cornell.edu/projects/btr/bioinformaticsschool/slides/gehrke.pdf](https://www.cs.cornell.edu/projects/btr/bioinformaticsschool/slides/gehrke.pdf)

[https://www.geeksforgeeks.org/tuple-relational-calculus-trc-in-dbms/](https://www.geeksforgeeks.org/tuple-relational-calculus-trc-in-dbms/)

[https://www.quora.com/Why-is-SQL-not-Turing-complete](https://www.quora.com/Why-is-SQL-not-Turing-complete)

### 2
[https://blog.ex-em.com/1683](https://blog.ex-em.com/1683)

### 3, 4, 5
[https://nbcamp.spartacodingclub.kr/blog/%EA%B0%9C%EB%85%90-%EC%BD%95-%EC%9B%B9-%EA%B0%9C%EB%B0%9C-%EC%A7%80%EC%8B%9D-%ED%8E%B8-ddl-dml-dcl-tcl-21313?utm_source=google&utm_medium=pmax&utm_campaign=nbc&utm_content=ai&utm_term=&gad_source=1&gclid=CjwKCAjw_ZC2BhAQEiwAXSgClhfjQHKBFxpGWiLsiGT6u0QUGNcdWCfOCE4SfhjWUqLo_hki1WiydhoC-eQQAvD_BwE](https://nbcamp.spartacodingclub.kr/blog/%EA%B0%9C%EB%85%90-%EC%BD%95-%EC%9B%B9-%EA%B0%9C%EB%B0%9C-%EC%A7%80%EC%8B%9D-%ED%8E%B8-ddl-dml-dcl-tcl-21313?utm_source=google&utm_medium=pmax&utm_campaign=nbc&utm_content=ai&utm_term=&gad_source=1&gclid=CjwKCAjw_ZC2BhAQEiwAXSgClhfjQHKBFxpGWiLsiGT6u0QUGNcdWCfOCE4SfhjWUqLo_hki1WiydhoC-eQQAvD_BwE)
