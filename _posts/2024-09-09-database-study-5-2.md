---
title: DB - 트랜잭션

author: milktea
date: 2024-09-10 03:00:00 +0800
categories: [Database]
tags: [Transaction]
pin: true
math: true
mermaid: true
---

> 데이터베이스 스터디 5주차에서 학습하고 정리한 내용입니다.
{: .prompt-info }


# 1. 트랜잭션

트랜잭션은 작업을 더이상 분리할 수 없는 논리적 작업 단위이다.
작업이 완전히 성공하던지 또는 원 상태로 복구하여 작업의 일부만 적용되는 부분 업데이트가 발생하지 않게 한다.

## 송금하기

트랜잭션에서 가장 많이 언급되는 예시는 은행 송금이다.
송금은 돈을 인출하여 받는 계좌에 입금하는 과정으로 구성된다.

1. A의 계좌에 100만원을 인출한다.
2. B의 계좌에 100만원을 입금한다.

만약에 쿼리 실행 도중 DB에 문제가 생겨 1의 과정만 실행되었다고 하자.
이 때 A의 계좌만 100만원이 차감되고 B의 계좌는 이전과 동일하다.

실제 서비스 시에는 엄청난 양의 데이터 조작이 수행되는데 이렇게 부분 업데이트 현상이 발생하면 데이터의 정합성을 맞추는데 상당히 어려워진다.
많은 RDBMS에서는 부분 업데이트 현상을 미리 방지하도록 완전 성공 또는 완전 실패시키는 방법으로 데이터 정합성을 유지한다.
이는 TCL(Transaction Control Language)인 BEGIN TRANSACTION, COMMIT, ROLLBACK으로 간단하게 구현할 수 있다.

## 커밋(Commit)

커밋은 논리적 작업들이 모두 정상적으로 처리되었다고 확정하는 명령이다.
커밋을 수행하면 변경 내역들이 DB에 영구 저장되고 트랜잭션 과정이 종료된다.

## 롤백(Rollback)

작업 중 문제가 발생하여 트랜잭션의 처리과정에서 발생한 모든 변경사항을 취소하는 명령이다.
롤백을 수행하면 트랜잭션 실행 이전으로 데이터가 돌아간다.

## autocommit 설정

- autocommit이 true라면?

autocommit 설정이 true가 되어 있으면 사용자가 명시적으로 트랜잭션을 지정하지 않은 경우 쿼리마다 커밋 작업을 수행한다.
이는 데이터 변경 작업이 즉시 DB에 반영되므로 Auto Commit 설정 활성화 시 데이터 일관성을 유지하기 어려울 수 있다.

- autocommit이 false라면?

autocommit 설정이 false일 때 사용자가 명시적으로 트랜잭션을 지정하지 않으면 첫 번째 쿼리 실행 시 트랜잭션이 자동으로 시작되어 COMMIT이나 ROLLBACK이 실행될 때까지 DB에 반영되지 않는다.

### JPA Hibernate에서 autocommit

JPA에서는 @Transactional 어노테이션을 많이 활용한다.
이 경우 Hibernate와 Spring에서 spring.datasource.hikari.auto-commit 설정이 true라면 JPA는 @Transactional 내의 쿼리를 실행하기 전에 `set autocommit = 0`을 먼저 실행하고 트랜잭션이 종료되면 `set autocommit = 1`을 다시 실행한다.

트랜잭션마다 set autocommit = 0/1이 중복해서 실행되므로 JPA에서는 spring.datasource.hikari.auto-commit을 false로 바꾸는 옵션이 추천된다.
자세한 튜닝 방법은 [여기](https://velog.io/@haron/Spring-set-autocommit-%EA%B0%9C%EC%84%A0%EC%9D%84-%ED%86%B5%ED%95%9C-%EC%84%B1%EB%8A%A5-%EC%B5%9C%EC%A0%81%ED%99%94%EB%A5%BC-%ED%95%B4%EB%B3%B4%EC%9E%90)에서 확인할 수 있다.

## 트랜잭션 주의할 점

트랜잭션은 꼭 필요한 최소의 코드에만 적용하는 것이 좋다.
다음과 같은 시나리오가 있다고 하자.

1. 처리 시작
2. 사용자의 로그인 여부 확인
3. 사용자의 글쓰기 내용의 오류 여부 확인
4. 첨부로 업로드된 파일 확인 및 저장
5. 사용자의 입력 내용을 DBMS에 저장
6. 첨부 파일 정보를 DBMS에 저장
7. 저장된 내용 또는 기타 정보를 DBMS에서 조회
8. 게시물 등록에 대한 알림 메일 발송
9. 알림 메일 발송 이력을 DBMS에 저장
10. 처리 완료

여기서 실질적으로 트랜잭션이 필요한 부분은 5, 6, 7, 9이다.
DBMS에 저장하지 않는 2~4를 트랜잭션에 포함하면 커넥션을 소유하는 시간이 길어져 사용 가능한 여유 커넥션 수가 줄어든다.

또한 **외부 시스템과 통신하는 8번 부분은 트랜잭션에 포함되면 안된다.**
외부 시스템이 응답을 하지 않는다면 응답이 올 때까지 트랜잭션을 유지하게 된다.
이 경우 트랜잭션과 연관된 잠금, 커넥션이 계속 유지되어 최악의 경우 데드락이 발생할 가능성이 있다.

따라서 이 시나리오에서 트랜잭션을 적용하는 좋은 방법 중 하나(요구사항에 따라 달라질 수 있다)는

1. 5~6을 하나의 트랜잭션
2. 9번을 하나의 트랜잭션

으로 묶는 방법이다. 7번은 트랜잭션 격리 수준에 따라 다를 수 있지만 반복 가능한 읽기(Repeatable Read)를 보장하려면 트랜잭션을 도입할 수 있다.
단순 1회성 조회는 생략할 수 있다.

---
# 2. ACID

트랜잭션이 데이터 정합성을 유지하기 위해 가져야 할 핵심적인 요소이다.
원자성(Atomicity), 일관성(Consistency), 격리성(Isolation), 지속성(Durability) 4가지 요소가 있다.

## 원자성

트랜잭션은 모두 성공하거나 모두 실패하여 부분 업데이트가 없어야 한다는 원칙이다.
개발자는 언제 커밋하고 언제 롤백할 지에 대한 기준을 세우고 코드를 작성해야 한다.

## 일관성

트랜잭션 이전과 이후에 데이터베이스는 항상 일관된 상태여야 한다는 규칙이다.
트랜잭션 실행 후에도 데이터에 모순이 없는 상태여야 한다.
위의 예시처럼 송금 전 후 부분 업데이트가 이뤄져 계좌의 총액이 일치하지 않는다면 일관성을 만족하지 않는다고 볼 수 있다.

## 격리성, 고립성

여러 트랜잭션이 동시에 실행될 때 다른 트랜잭션에 영향을 받거나 주면 안된다.
여러 트랜잭션이 동시에 실행될 때 하나의 트랜잭션이 먼저 종료되면 트랜잭션 실행 중 DB의 데이터가 변할 수 있다.
이 때 트랜잭션이 종료되도 영향을 받지 않도록 DBMS는 여러 종류의 격리 수준을 제공한다.

### 영속성

트랜잭션이 커밋되면 데이터베이스에 영구적으로 저장되어야 한다.


---
# Reference
[https://velog.io/@syh0397/10.-%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%98-BEGIN-COMMIT-ROLLBACK](https://velog.io/@syh0397/10.-%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%98-BEGIN-COMMIT-ROLLBACK)

[https://velog.io/@haron/Spring-set-autocommit-%EA%B0%9C%EC%84%A0%EC%9D%84-%ED%86%B5%ED%95%9C-%EC%84%B1%EB%8A%A5-%EC%B5%9C%EC%A0%81%ED%99%94%EB%A5%BC-%ED%95%B4%EB%B3%B4%EC%9E%90](https://velog.io/@haron/Spring-set-autocommit-%EA%B0%9C%EC%84%A0%EC%9D%84-%ED%86%B5%ED%95%9C-%EC%84%B1%EB%8A%A5-%EC%B5%9C%EC%A0%81%ED%99%94%EB%A5%BC-%ED%95%B4%EB%B3%B4%EC%9E%90)

[https://engineerinsight.tistory.com/210](https://engineerinsight.tistory.com/210)

백은빈, 이성욱 편저, Real MySQL 8.0, 위키북스
