---
title: DB - 참조 무결성과 MySQL 외래키 제약 조건
author: milktea
date: 2024-08-21 10:00:00 +0800
categories: [Database]
tags: [Integrity]
pin: true
math: true
mermaid: true
---

> 데이터베이스 스터디 2주차에서 학습하고 정리한 내용입니다.
{: .prompt-info }

# 1. 참조 무결성
타 테이블과 연관된 데이터가 입력, 수정, 삭제 시에도 데이터 간에 정확한 참조 관계를 유지시킨다.

외래키에 적용되는 개념으로 외래키가 어떠한 데이터를 참조할 때 이 데이터는 반드시 존재하는 값이어야 한다.
외래키가 유효하지 않은 데이터를 참조한다면 데이터의 일관성에 문제가 생길 수 있다.
이러한 참조 무결성을 만족하기 위해 여러 관계형 데이터베이스에서 외래키 제약 조건을 지원한다.

예를 들면 사용자가 리뷰를 삭제한다고 하자.
이 리뷰에는 사용자가 함께 올린 이미지가 있다.
이때 리뷰를 삭제하고 이미지의 데이터는 변하지 않는다면 이미지는 유효하지 않은 리뷰를 참조하므로 이 경우 참조 무결성이 깨질 수 있다.

## MySQL의 외래키 제약 조건
MySQL에서는 `RESTRICT`, `CASCADE`, `SET NULL`, `NO ACTION` 옵션이 있다.
SET DEFAULT 옵션은 SQL 표준에는 존재하지만 InnoDB와 NDB 스토리지 엔진에서 지원하지 않는다.
MySQL은 별다른 외래키 제약 조건을 지정하지 않으면 `RESTRICT`나 `NO ACTION`을 적용한다. 

### 1) RESTRICT
```sql
ALTER TABLE review_images
ADD CONSTRAINT fk_review_images
FOREIGN KEY (review_id) REFERENCES reviews(id)
ON DELETE RESTRICT
ON UPDATE RESTRICT;
```

부모 테이블에 있는 데이터의 삭제나 변경을 막는다.
이 경우 부모 테이블을 참조하는 데이터를 모두 삭제해야 부모 테이블의 데이터를 삭제할 수 있다.

### 2) CASCADE
```sql
ALTER TABLE review_images
ADD CONSTRAINT fk_review_images
FOREIGN KEY (review_id) REFERENCES reviews(id)
ON DELETE CASCADE
ON UPDATE CASCADE;
```

부모 테이블에서 행의 삭제나 변경이 일어났을 때 자식 테이블에서 자동으로 삭제나 변경이 일어난 데이터를 참조하는 행을 삭제한다.
`ON DELETE CASCADE`는 참조하는 행이 삭제되면 자동으로 삭제하고 `ON UPDATE CASCADE`는 참조하는 행의 기본키 등이 변경되면 그 기본키를 참조하는 리뷰 테이블의 외래키 값도 자동으로 업데이트한다.

만약 두 테이블이 서로를 참조하는 관계라면 양쪽 모두에서 `ON DELETE CASCADE` 또는 `ON UPDATE CASCADE`를 설정해야만 올바르게 작동한다.
그렇지 않으면 CASCADE 작업이 실패하게 된다. 

### 3) SET NULL
```sql
ALTER TABLE review_images
ADD CONSTRAINT fk_review_images
FOREIGN KEY (review_id) REFERENCES Reviews(id)
ON DELETE SET NULL
ON UPDATE SET NULL;
```

부모 테이블에서 행의 삭제나 변경이 일어날 때 자식 테이블의 외래키 값을 NULL로 변경한다.
**SET NULL 외래 키 제약 조건이 정상적으로 적용되기 위해서는 외래키에 NOT NULL을 선언하면 안된다.**

### 4) NO ACTION
SQL 표준에서 존재하는 키워드이다.
InnoDB에서 이 키워드는 RESTRICT와 동일하게 삭제, 변경이 일어날 때 즉각적으로 무결성을 검사한다.
하지만 NDB 스토리지 엔진에서는 **외래 키 무결성 검사가 부모 테이블에서 삭제나 업데이트 시점이 아닌, 커밋할 때까지 지연된다.**
따라서 NDB 스토리지 엔진에서는 커밋 시점에만 무결성을 확보하면 되기 때문에 유연하게 처리할 수 있다.

## SOFT DELETE
실제로 데이터를 삭제하지 않고 상태만 '삭제한 상태'로 변경하는 방법이다.
사용자에게는 삭제한 데이터를 보여주지 않지만 여전히 데이터베이스에 존재한다.
이 방법으로도 참조 무결성을 유지할 수 있다.

### 장점
1. 데이터 복구가 쉽다. 상태만 바꾸면 된다.
2. 데이터의 무결성을 유지한다. 삭제된 상태일 뿐 데이터는 남아 있으므로 여전히 참조할 수 있다.
3. 데이터의 변경 이력을 추적하기 쉽다. 데이터가 남아 있기 때문에 언제 어떤 데이터가 삭제되었는지 추적이 쉽다.

### 단점
1. 데이터베이스 용량이 커진다. 삭제된 상태라도 여전히 데이터베이스에 남아있기 때문에 저장 공간을 낭비할 수도 있다.
2. 쿼리가 복잡해진다. 삭제되지 않은 상태 만을 조회하기 위한 추가적인 조건이 필요하다.
3. 성능이 저하될 수 있다. 데이터베이스 용량이 커지므로 조회 시 성능이 감소할 수 있고 인덱스 활용이 어려워질 수 있다.

결론적으로, 참조 무결성을 충족하기 위해 SOFT DELETE도 고려할 수 있으나 불필요한 리소스를 잡아먹고 성능이 저하될 수 있음을 인지해야 한다.
중요한 데이터이거나 일관성이 중요한 데이터일 경우 SOFT DELETE를, 데이터의 수가 아주 많은 경우 완전 삭제를 고려할 수 있다.

이 때 삭제한 데이터만을 보관하는 **Archiving Table**을 도입하여 성능 저하 문제를 해결할 수 있지만 추가적인 아카이빙 작업이 필요하다는 단점도 있다.

---
# Reference
홍봉희 편저, 데이터베이스 SQL 프로그래밍 'MySQL 실습', 부산대학교출판문화원

### 1
[https://dev.mysql.com/doc/refman/8.4/en/create-table-foreign-keys.html](https://dev.mysql.com/doc/refman/8.4/en/create-table-foreign-keys.html)

[https://velog.io/@heypop/2023.10.22-Soft-Delete-Hard-Delete](https://velog.io/@heypop/2023.10.22-Soft-Delete-Hard-Delete)

[https://stackoverflow.com/questions/21175228/sql-database-best-practices-use-of-archive-tables](https://stackoverflow.com/questions/21175228/sql-database-best-practices-use-of-archive-tables)


