---
title: DB - 정규화

author: milktea
date: 2024-09-03 14:00:00 +0800
categories: [Database]
tags: [NormalForm]
pin: true
math: true
mermaid: true
---

> 데이터베이스 스터디 4주차에서 학습하고 정리한 내용입니다.
{: .prompt-info }


# 1. 정규화(Normalization)

정규화는 데이터의 중복을 제거하거나 최소화하고 데이터 종속이 논리적으로 표현되도록 데이터를 재구성하는 과정이다.
이 과정을 수행함으로써 데이터 이상 현상이 제거된다.

---
# 2. 제 1 정규형(First Normal Form, 1NF)

> 제 1 정규형은 릴레이션 내 모든 데이터가 스칼라 값만 가지고 데이터 그룹을 속성으로 가질 수 없다.
{: .prompt-info }

## 제 1 정규형 위반 사례

- 제 1 정규형을 위반한 첫 번째 사례(student 테이블)

| student_id | Subject_name       |
|------------|--------------------|
| 1          | Algorithm, Network |
| 2          | Database           |
| 3          | Network, Database  |

- 제 1 정규형을 위반한 두 번째 사례(student 테이블)

| student_id | subject_name_1 | subject_name_2 |
|------------|----------------|----------------|
| 1          | Algorithm      | Network        |
| 2          | Database       | None           |
| 3          | Network        | Database       |

위의 두 테이블은 모두 제 1 정규형을 위반하고 있다.
이렇게 테이블을 구성했을 때 문제점은 다음과 같다.

1. 데이터 조회 시 복잡하다. 특히 첫 번째 테이블의 경우 추가적인 문자열 파싱이 필요할 수 있다.
2. 유연성이 떨어진다. 특히 두 번째 테이블의 경우 수업을 3개 이상 듣는 학생이 있다면 새로운 컬럼을 추가하는 작업이 필요하다.
3. 삽입 이상이 발생한다. 특히 두 번째 테이블의 경우 2번 튜플처럼 듣는 과목이 하나일 때 'None'이라는 불필요한 값을 추가해야 한다.
4. 갱신 이상이 발생한다. Network을 Computer Network로 변경하고 싶다면 모든 튜플들 내 Network을 변경하지 않으면 데이터가 불일치 되는 문제가 발생한다.
5. 삭제 이상이 발생한다. 1번 학생을 삭제할 때 Algorithm이란 과목도 함께 삭제가 된다.

이 제 1 정규형을 만족하지 않을 때 가장 큰 문제는 **데이터를 조회하고 조작하기 어렵다**는 점이다.

## 제 1 정규형 도입

- 제 1 정규형을 도입한 student 테이블

| student_id | subject_name |
|------------|--------------|
| 1          | Algorithm    |
| 1          | Network      |
| 2          | Database     |
| 3          | Network      |
| 3          | Database     |

1 정규형을 도입하면 문자열 처리 등의 과정이 생략되어 데이터 조회와 조작이 간편해진다.
하지만 1 정규형을 도입한 것 만으로는 이상 현상이 해결되지 않아 추가적인 정규화 과정이 필요하다.

---

# 3. 제 2 정규형(2NF)

> 제 2 정규형은 제 1 정규형을 만족하면서 후보 키가 아닌 모든 속성이 후보 키에 대해 완전 함수적 종속이어야 한다.
{: .prompt-info }

## 제 2 정규형 위반 사례

- 제 2 정규형을 위반한 student 테이블

| student_id | student_name | subject_id | subject_name | score |
|------------|--------------|------------|--------------|-------|
| 1          | Kim          | 1          | Database     | A0    |
| 1          | Kim          | 2          | Algorithm    | B0    |
| 2          | Park         | 2          | Algorithm    | A0    |
| 2          | Park         | 3          | Network      | B+    |

이 경우 student_id, subject_id -> subject_name을 만족하지만 student_id를 제외한 subject_id -> subject_name 또한 함수 종속성을 만족한다.
이 때 subject_name이라는 후보 키가 아닌 속성이 후보 키 (student_id, subject_id)에 대해 **부분 함수적 종속을 만족하므로 제 2 정규형을 위반**한다.
이렇게 테이블을 구성하면 제 1 정규형에 비해 데이터를 조회하고 조작하기는 쉬워졌지만 여전히 여러 이상 현상들이 나타날 수 있다.

1. 삽입 이상이 발생한다. 아직 학생이 없는 과목을 개설할 때 학생 정보에 의미없는 값을 추가해야 한다.
2. 갱신 이상이 발생한다. Algorithm의 과목 이름을 변경할 때 모든 이름을 변경하지 않으면 데이터 불일치가 발생한다.
3. 삭제 이상이 발생한다. Kim의 데이터를 삭제할 때 Database에 대한 정보가 함께 삭제될 수 있다.

## 제 2 정규형 도입

- student 테이블

| student_id | student_name |
|------------|--------------|
| 1          | Kim          |
| 2          | Park         |

- subject 테이블

| subject_id | subject_name |
|------------|--------------|
| 1          | Database     |
| 2          | Algorithm    |
| 3          | Network      |

- score 테이블

| student_id | subject_id | score |
|------------|------------|-------|
| 1          | 1          | A0    |
| 1          | 2          | B0    |
| 2          | 2          | A0    |
| 2          | 3          | B+    |

제 2 정규형을 도입한 결과 중복되는 데이터를 제거하여 기존 테이블의 갱신 이상, 삽입 이상 및 삭제 이상 문제를 해결할 수 있다.
하지만 제 2 정규형은 특정한 상황에서 여전히 이상 현상이 발생할 수 있다.
이를 해결하기 위해 제 3 정규형을 도입할 수 있다.

---
# 4. 제 3 정규형(3NF)

> 제 3 정규형은 제 2 정규형을 만족하면서 후보 키가 아닌 모든 속성이 후보 키에 대하여 이행적 함수 종속성이 없어야 한다.
{: .prompt-info }

## 제 3 정규형 위반 사례

- address 테이블

| id | state | city          | street_name         |
|----|-------|---------------|---------------------|
| 1  | NY    | New York City | Broadway            |
| 2  | CA    | Los Angeles   | Sunset Boulevard    |
| 3  | IL    | Chicago       | Michigan Avenue     |
| 4  | TX    | Houston       | Westheimer Road     |
| 5  | CA    | Los Angeles   | Hollywood Boulevard |
| 6  | NY    | New York City | Fifth Avenue        |
| 7  | CA    | San Francisco | Lombard Street      |

제 2 정규형을 도입하더라도 테이블 내 이행적 함수 종속성이 있는 경우 여전히 이상 현상이 발생할 수 있다.
이 테이블은 기본 키가 단일 속성이므로 기본 키의 부분 집합이 존재하지 않는다.
따라서 기본 키에 대한 부분 함수적 종속은 없으므로 제 2 정규형이 적용되어 있음을 확인할 수 있다.

이 때 id -> city, city -> state 함수 종속성을 만족하면서 id -> state 함수 종속성을 만족한다.
이 경우 기본 키가 아닌 속성인 state와 city가 기본 키에 대해 이행적 함수 종속성을 만족한다.
이행적 함수 종속성은 키가 아닌 속성들 간의 종속성이 존재하므로 데이터가 중복될 가능성이 높다.

1. 삽입 이상이 발생한다. state와 city만 존재하는 데이터를 넣을 때 street_name에 의미 없는 값을 추가해야 한다.
2. 갱신 이상이 발생한다. 한 city의 이름이 변경될 때 모든 이름을 변경하지 않으면 데이터 불일치가 발생한다.

이와 같이 이상 현상이 발생할 수 있으므로 정규형을 통해 이행적 함수 종속성을 제거해야 한다.

## 제 3 정규형 도입

- state 테이블

| id | state |
|----|-------|
| 1  | NY    |
| 2  | CA    |
| 3  | IL    |
| 4  | TX    |

- city 테이블

| id | state_id | city_name     |
|----|----------|---------------|
| 1  | 1        | New York City |
| 2  | 2        | Los Angeles   |
| 3  | 2        | San Francisco |
| 4  | 3        | Chicago       |
| 5  | 4        | Houston       |

- street 테이블

| id | city_id | street_name         |
|----|---------|---------------------|
| 1  | 1       | Broadway            |
| 2  | 1       | Fifth Avenue        |
| 3  | 2       | Sunset Boulevard    |
| 4  | 2       | Hollywood Boulevard |
| 5  | 3       | Lombard Street      |
| 6  | 4       | Michigan Avenue     |
| 7  | 5       | Westheimer Road     |

이렇게 이행적 함수 종속성을 제거하는 과정으로 제 3 정규형을 만족하도록 할 수 있다.
제 3 정규형을 도입하기 전과 달리 데이터 중복이 존재하지 않아 갱신 이상이 발생하지 않는다.
또한 삽입 이상도 발생하지 않는다.

하지만 제 3 정규형도 아주 특이한 상황에서 이상 현상이 발생할 수도 있다.
이 때 이상 현상을 제거하기 위해 BCNF, 4NF, 5NF를 도입할 수 있지만 정규화가 고도화될 수록 성능이 저하될 수 있으므로 이를 고려하여 도입해야 한다.
또한 성능과 데이터 무결성의 이점을 비교하여 성능이 더 우선시 될 경우 제 3 정규형 이전의 정규형도 의도적으로 위반할 수 있다.

---
# Reference
홍봉희 편저, 데이터베이스 SQL 프로그래밍 'MySQL 실습', 부산대학교출판문화원
