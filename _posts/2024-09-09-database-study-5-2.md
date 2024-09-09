---
title: 데이터베이스 스터디 5주차 - 트랜잭션

author: milktea
date: 2024-09-02 14:00:00 +0800
categories: [Database]
tags: [Transaction]
pin: true
math: true
mermaid: true
---
# 1. 트랜잭션

트랜잭션은 작업의 완전성을 보장해주는 것이다.
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
이는 데이터 변경 작업이 즉시 DB에 반영되므로 Auto Commit 설정 활성화 시 데이터 정합성이 위배되는 부분을 주의해야 한다.

- autocommit이 false라면?

autocommit 설정이 false일 때 사용자가 명시적으로 트랜잭션을 지정하지 않으면 첫 번째 쿼리 실행 시 트랜잭션이 자동으로 시작되어 COMMIT이나 ROLLBACK이 실행될 때까지 DB에 반영되지 않는다.

### JPA Hibernate에서 autocommit

JPA에서는 @Transactional 어노테이션을 많이 활용한다.
이 경우 Hibernate에서는 autocommit 설정이 true라면 JPA는 @Transactional 내의 쿼리를 실행하기 전에 `set autocommit = 0`을 먼저 실행하고 트랜잭션이 종료되면 `set autocommit = 1`을 다시 실행한다.

트랜잭션마다 set autocommit = 0/1이 중복해서 실행되므로 JPA에서는 autocommit을 false로 바꾸는 옵션이 추천된다.

## 트랜잭션 주의할 점

트랜잭션은 

---
# 2. ACID


---
# Reference
[https://velog.io/@syh0397/10.-%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%98-BEGIN-COMMIT-ROLLBACK](https://velog.io/@syh0397/10.-%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%98-BEGIN-COMMIT-ROLLBACK)

[https://velog.io/@haron/Spring-set-autocommit-%EA%B0%9C%EC%84%A0%EC%9D%84-%ED%86%B5%ED%95%9C-%EC%84%B1%EB%8A%A5-%EC%B5%9C%EC%A0%81%ED%99%94%EB%A5%BC-%ED%95%B4%EB%B3%B4%EC%9E%90](https://velog.io/@haron/Spring-set-autocommit-%EA%B0%9C%EC%84%A0%EC%9D%84-%ED%86%B5%ED%95%9C-%EC%84%B1%EB%8A%A5-%EC%B5%9C%EC%A0%81%ED%99%94%EB%A5%BC-%ED%95%B4%EB%B3%B4%EC%9E%90)

백은빈, 이성욱 편저, Real MySQL 8.0, 위키북스
