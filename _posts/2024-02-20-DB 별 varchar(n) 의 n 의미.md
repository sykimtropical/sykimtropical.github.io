---
layout: post
title:  "DB별 varchar(n)의 n 의미"
date:   2024-02-20 10:00:00 +0900
category: [DB]
tags: [DB,oracle,mysql,mariadb]
lastmod : 2024-02-20 10:00:00 +0900
sitemap :
  changefreq : daily
  priority : 1.0
---
# Oracle

`varchar2(10)` 은 10 byte를 의미한다.<br>
oracle에서 영어는 1byte, 한글은 2byte 이기 때문에 그동안 막연하게 한글이 들어가는 컬럼은 100자 -> varchar2(200) 으로 생성해왔다.<br>
<br>

```sql
-- 테스트용 테이블 생성
CREATE TABLE test(
	name varchar2(10)
);

-- 한글 10글자를 INSERT
INSERT INTO test values('일이삼사오육칠팔구십');

Query execution failed

Reason:
SQL Error [12899] [72000]: ORA-12899: value too large for column
"TEST"."NAME" (actual: 20, maximum: 10)
-- 10byte 컬럼에 20byte가 들어왔다는 오류 확인
```
<br><br>

지금까지 잘못 알고 있었다.<br>

```sql
-- 테스트용 테이블 생성, 아까와 달리 10 char 로 생성하였다.
CREATE TABLE test(
	name varchar2(10 char)
);

-- 한글 10글자를 insert 하였을때 성공 하는것을 확인 할 수 있다.
INSERT INTO test values('일이삼사오육칠팔구십');

SELECT name FROM test;
-- '일이삼사오육칠팔구십' 이 조회되는 것을 확인 할 수 있다.

```

`varchar2(n char)` 에서 `char`는 byte 와 상관 없이 글자수(length) 인 것을 확인 할 수 있다.<br><br>


# Mysql & MariaDB

oracle에서 `varchar2(n)` 의 n은 byte 수 인 것을 보고 당연히 모든 DB 는 다 그럴 줄 알고 똑같이 사용해왔다.<br>
찾아보니 mysql 4.1 이상부터는 전부 기본적으로 lenght 라는 내용을 볼 수 있었다.<br>
<br>
Mysql, MariaDB 전부 확인한 결과 바이트와 상관없이 길이로 insert가 되었다.<br>
<br>

```sql
CREATE TABLE test(
	name varchar(10)
);

-- 한글 10글자를 insert 하였을때 성공 하는것을 확인 할 수 있다.
INSERT INTO test values('일이삼사오육칠팔구십');

SELECT name FROM test;
-- '일이삼사오육칠팔구십' 이 조회되는 것을 확인 할 수 있다.
```


