---
title: 데이터베이스 스터디 4주차 - 이상 현상

author: milktea
date: 2024-09-01 13:00:00 +0800
categories: [Database]
tags: [Anomaly]
pin: true
math: true
mermaid: true
---
# 1. 이상 현상

이상 현상은 불필요한 데이터 중복으로 인해 릴레이션에 대한 데이터 삽입, 수정, 삭제 연산을 할 때 발생할 수 있는 부작용이다.
이상 현상이 발생하게 되면 데이터 무결성이 위배되어 데이터가 중복되거나 불일치가 될 수 있다.
이 이상 현상을 제거하기 위해 정규화가 필요하다.
아래와 같은 예시 테이블에서 어떠한 이상 현상들이 발생하는지 살펴보자.

| Student_id | Student_name | Subject_id | Subject_name | Score |
|------------|--------------|------------|--------------|-------|
| 1          | Kim          | 1          | Network      | A0    |
| 1          | Kim          | 2          | Algorithm    | B0    |
| 2          | Lee          | 2          | Algorithm    | A0    |
| 2          | Lee          | 3          | Database     | B+    |
| 3          | Park         | 1          | Network      | A+    |
| 3          | Park         | 3          | Database     | B0    |


## 1) 삽입 이상

이 테이블에서 Operation System 과목이 새로 개설되었다고 하자.
하지만 이 때 신규 개설 과목이라 이 수업을 듣는 학생들이 없는데 데이터를 어떻게 추가해야 할까?
학생에 '학생 없음'과 같은 의미 없는 값을 추가해야 할 것이다.
이와 같이 삽입 이상은 **데이터 삽입 시 의도와 다른 값들도 삽입되는 현상**이다.

| Student_id | Student_name | Subject_id | Subject_name     | Score |
|------------|--------------|------------|------------------|-------|
| 1          | Kim          | 1          | Network          | A0    |
| 1          | Kim          | 2          | Algorithm        | B0    |
| 2          | Lee          | 2          | Algorithm        | A0    |
| 2          | Lee          | 3          | Database         | B+    |
| 3          | Park         | 1          | Network          | A+    |
| 3          | Park         | 3          | Database         | B0    |
| ?          | ?            | 4          | Operation System | ?     |

## 2) 갱신 이상

이 테이블에서 Algorithm이란 과목이 Data Structure And Algorithm이란 이름으로 변경되었다고 하자.
이 경우 Subject_name이 Algorithm인 모든 튜플의 이름을 변경해야 한다.
**만약 일부 튜플들의 이름만 변경되었다면 동일한 Subject_id 임에도 데이터가 불일치되는 상황**이 발생한다.
이와 같이 갱신 이상은 **속성값 갱신 시 일부 튜플들만 갱신되어 모순이 발생하는 현상**이다.

| Student_id | Student_name | Subject_id | Subject_name                           | Score |
|------------|--------------|------------|----------------------------------------|-------|
| 1          | Kim          | 1          | Network                                | A0    |
| 1          | Kim          | 2          | Data Structure And Algorithm -> 불일치 | B0    |
| 2          | Lee          | 2          | Algorithm -> 불일치                    | A0    |
| 2          | Lee          | 3          | Database                               | B+    |
| 3          | Park         | 1          | Network                                | A+    |
| 3          | Park         | 3          | Database                               | B0    |

## 3) 삭제 이상

이 테이블에서 Kim과 Park에 대한 정보만을 삭제해야 한다고 하자.
이 때 Kim과 Park에 대한 정보를 삭제하면 Network 과목에 대한 정보도 함께 사라지게 된다.

| Student_id | Student_name | Subject_id | Subject_name | Score |
|------------|--------------|------------|--------------|-------|
| 2          | Lee          | 2          | Algorithm    | A0    |
| 2          | Lee          | 3          | Database     | B+    |

이와 같이 삭제 이상은 **튜플 삭제로 인해 다른 데이터까지 함께 삭제되는 현상**이다.

## 정규화로 이상 현상 해결하기

정규화를 진행하면 아래와 같이 세 개의 테이블로 분리할 수 있다.

- Student 테이블

| Student_id | Student_name |
|------------|--------------|
| 1          | Kim          |
| 2          | Lee          |
| 3          | Park         |

- Subject 테이블

| Subject_id | Subject_name |
|------------|--------------|
| 1          | Network      |
| 2          | Algorithm    |
| 3          | Database     |

- Score 테이블

| Student_id | Subject_id | Score |
|------------|------------|-------|
| 1          | 1          | A0    |
| 1          | 2          | B0    |
| 2          | 2          | A0    |
| 2          | 3          | B+    |
| 3          | 1          | A+    |
| 3          | 3          | B0    |

### 삽입 이상의 발생 확인하기

Operation System이란 과목이 새로 개설되었고 아직 학생이 없을 때 Subject 테이블에만 추가할 수 있다.
이 경우 과목에 대한 정보 외 의도하지 않은 정보를 추가하지 않는다.

### 갱신 이상의 발생 확인하기

Algorithm 과목이 Data Structure And Algorithm 과목으로 이름이 변경되었을 때 Subject 테이블의 컬럼 하나만 변경하면 된다.
컬럼 하나만 변경하면 되므로 일부만 변경해서 데이터가 불일치하는 경우는 없다.

### 삭제 이상의 발생 확인하기

Kim과 Park에 대한 정보를 삭제할 때 Student 테이블과 Score 테이블에서만 삭제하면 된다.
따라서 의도하지 않은 Subject 데이터가 삭제되는 경우가 없다.
이 때 학생이 삭제될 때 학생의 점수도 함께 삭제되는 과정은 의도하지 않은 삭제로 볼 수 없다.
Student_id와 Subject_id를 외래키로 두고 Student_id가 삭제될 때 점수도 함께 삭제되어 참조 무결성이 보장된다.

---
# Reference
https://velog.io/@sukyeongs/Database-Anomaly

