---
title: DB - 함수 종속성

author: milktea
date: 2024-09-02 14:00:00 +0800
categories: [Database]
tags: [FunctionalDependency]
pin: true
math: true
mermaid: true
---

> 데이터베이스 스터디 4주차에서 학습하고 정리한 내용입니다.
{: .prompt-info }

# 1. 함수 종속성(Functional Dependency)

정의는 다음과 같다.

> X⊆R, Y⊆R 일 때 relation R의 tuple t1, t2 에 대하여 (if (t1[X] = t2[X]) 이면 -> (t1[Y] = t2[Y]))를 만족할 때 “Y는 X에 함수적으로 종속되었다"고 한다.
{: .prompt-info }

수학에서 X^2 + Y^2 = 1은 함수로 표현하지 않는다.
f(0) = 1도 되면서 f(0) = -1도 되기 때문이다.

마찬가지로 X 속성의 값이 동일할 때 Y 속성의 값도 동일하다면 "Y는 X에 함수적으로 종속되었다"고 표현한다.
**하나의 X 속성 값에서 두 개의 Y 속성 값이 나올 수 없다**는 말이다.
이 때 X와 Y는 하나의 속성일 수도, 또는 여러 개의 속성들의 집합일 수도 있다.

- student 테이블

| student_id | student_name | subject_name | score |
|------------|--------------|--------------|-------|
| 1          | Kim          | Database     | A0    |
| 1          | Kim          | Algorithm    | B0    |
| 2          | Park         | Algorithm    | A0    |
| 2          | Park         | Network      | B+    |

## 1) student_id -> student_name

학번마다 하나의 이름만을 가지므로 함수 종속성을 만족한다.

## 2) student_id, subject_name -> score

학번과 과목 모두 일치할 때 성적이 다른 경우는 없다.(한 학생이 동일한 과목을 두 번 들어서 성적이 다른 경우가 없다)
따라서 `student_id, subject_name -> score` 함수 종속성을 만족한다.
이 경우 특정 학생이 특정 과목에서 받은 성적은 유일하게 하나로 결정된다.

## 3) student_id -x-> score

하나의 학번에서 다른 성적 값이 나올 수 있다.
예를 들어 학번이 1인 학생은 A0 score도 가지고 B0 score도 가진다.
따라서 함수 종속성을 만족한다고 볼 수 없다.

---
# 2. 완전 함수 종속(Full FD)

정의는 다음과 같다.

> Y가 X에 종속되어 있으면서 X의 어떤 진부분 집합에도 종속되지 않을 때 Y는 X에 완전 함수적 종속되어 있다고 본다.
{: .prompt-info }

Y가 X가 아닌 부분 집합에 종속되면 안된다는 말이다.
**X는 함수 종속성을 만족하는 최소 컬럼만을 포함해야 완전 함수 종속**으로 볼 수 있다.

- student 테이블

| student_id | student_name | subject_name | score |
|------------|--------------|--------------|-------|
| 1          | Kim          | Database     | A0    |
| 1          | Kim          | Algorithm    | B0    |
| 2          | Park         | Algorithm    | A0    |
| 2          | Park         | Network      | B+    |

## 부분 함수 종속(Partial FD)
Y가 X 뿐만 아니라 X의 부분 집합에도 종속되는 경우를 부분 함수적 종속이라고 한다.

## student_id, student_name, subject_name -> score

student_id, student_name, subject_name 조합으로 score을 유일하게 결정할 수 있다.
하지만 student_id, subject_name 조합이나 student_name, subject_name 조합으로도 score을 유일하게 결정할 수 있다.

이 경우 Y(score)가 X(student_id, student_name, subject_name)에 종속되어 있으면서 Y는 X의 부분 집합인 (student_id, subject_score)에도 종속되어 있다.
따라서 **Y는 X와 X의 진부분 집합에도 종속되어 있으므로 Y는 X에 부분 함수 종속이다.**

## student_id, subject_name -> score

student_id 만으로는 score를 결정할 수 없고 subject_name 만으로도 score를 결정할 수 없다.
따라서 이 경우 Y(score)는 X(student_id, subject_name)에 종속되어 있으면서 Y는 X의 부분 집합인 student_id이나 subject_name에 종속되지 않는다.
따라서 **Y는 X의 어떠한 진부분 집합에도 종속되어 있지 않고 X에만 종속되어 있으므로 Y는 X에 완전 함수 종속이다.**

---
# 3. 이행적 함수 종속(Transitive FD)

이행적 함수적 종속의 정의는 다음과 같다.

> X -> Y, Y -> Z가 성립할 때 X -> Z가 성립하면 이행적 함수 종속이 된다.
{: .prompt-info }

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

## street_name -> city, city -> state, street_name -> state

street_name으로 city를 유일하게 결정할 수 있다.
그러므로 street_name -> city 함수 종속성을 만족한다.
또한 city -> state, street_name -> state도 함께 함수 종속성을 만족한다.

이 경우 street_name -> city, city -> state 이면서 street_name -> state이므로 이행적 함수 종속을 만족한다.

---
# Reference
[http://contents.kocw.or.kr/document/lec/2011_2/dunksung/ParkUchang/08.pdf](http://contents.kocw.or.kr/document/lec/2011_2/dunksung/ParkUchang/08.pdf)
