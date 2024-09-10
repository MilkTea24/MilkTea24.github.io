---
title: 데이터베이스 스터디 5주차 - DB 세션

author: milktea
date: 2024-09-09 14:00:00 +0800
categories: [Database]
tags: [Session]
pin: true
math: true
mermaid: true
---
# 1. DB 세션과 커넥션

많은 데이터베이스 관리 시스템(DBMS)는 SQL 서버와 어플리케이션이 모듈로 존재한다.
따라서 어플리케이션이 SQL 서버에서 데이터를 가지고 오기 위해서는 네트워크 연결이 필요하다.
이 때 **어플리케이션과 SQL 서버 사이 물리적 연결을 커넥션**이라고 한다.

커넥션을 연결한 후 SQL 서버에서는 어플리케이션의 정보를 가지고 있어 어플리케이션을 식별한다.
이 때 **SQL 서버에서 어플리케이션 정보, 환경 설정등을 저장하는 저장소를 세션**이라고 한다.

## MySQL에서의 DB 세션과 커넥션

위의 설명은 일반적인 예시이고 커넥션과 세션의 아키텍처와 세부 구현은 DBMS마다 상이하다.
MySQL 기준으로 세션과 커넥션에 대해 알아보겠다.

### 커넥션

MySQL 서버는 프로세스 기반이 아닌 스레드 기반으로 작동하며 크게 포그라운드와 백그라운드 스레드로 구분할 수 있다.

![img.png](/assets/img/posts/database/study-5-1/img.png)

**사용자와 MySQL 서버는 포그라운드 스레드를 통해 커넥션을 구축**한다.
이 포그라운드 스레드는 각 사용자가 요청하는 쿼리 문장을 처리한다.

사용자와 MySQL 서버 간의 커넥션이 종료되면 해당 커넥션을 담당하는 스레드는 다시 스레드 캐시(Thread Cache)로 되돌아간다.
스레드 캐시는 클라이언트 커넥션 시 매번 새로운 포그라운드 스레드를 생성하지 않고 재활용할 수 있는 기능을 제공한다.

### 세션

![img_1.png](/assets/img/posts/database/study-5-1/img_1.png)

MySQL에서 사용되는 메모리 영역은 크게 글로벌 메모리 영역과 로컬 메모리 영역으로 구분할 수 있다.
글로벌 메모리 영역은 MySQL 전체 서버를 유지하기 위한 데이터를 저장하고 있다.
반면 로컬 메모리 영역은 **세션 메모리 영역이라고도 하며 클라이언트 포그라운드 스레드가 쿼리를 처리하는데 사용되는 메모리 영역**이다.

이 세션 메모리 영역은 각 클라이언트 스레드별로 독립적으로 할당되어 스레드 간 절대 공유되지 않는다.
따라서 세션 메모리 영역으로 하나의 커넥션 내에서 독립적인 데이터 저장 및 환경 설정을 할 수 있다.

---
# Reference
[https://dba.stackexchange.com/questions/13698/what-is-the-difference-between-a-connection-and-a-session](https://dba.stackexchange.com/questions/13698/what-is-the-difference-between-a-connection-and-a-session)

백은빈, 이성욱 편저, Real MySQL 8.0, 위키북스
