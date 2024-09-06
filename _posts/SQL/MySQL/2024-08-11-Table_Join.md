---
title:  "[MySQL] 테이블 INNER JOIN"
excerpt: ""

categories: [SQL, MySQL]
tags: [SQL, MySQL, JOIN, INNER]

toc: true
toc_sticky: true
 
date: 2024-08-11
last_modified_at: 2024-08-11
---

RDBMS의 한 종류인 MySQL 에서 2개 이상의 테이블을 JOIN 하려고 할 때, 몇 가지 규칙이 존재합니다.  

이번 포스트에서는 MySQL 에서 사용하는 INNER JOIN 절에 대해 알아보겠습니다.  

> INNER JOIN 절에서 INNER 을 **생략**하여 JOIN 만 사용할 수 있습니다.  

## 테이블들의 교집합

**JOIN 절은 2개 이상의 테이블에서 교집합을 이용해 원하는 데이터를 추출하는 연산**입니다.  

이때, JOIN 을 사용하기 위해서는 **ON** 절을 함께 사용해야 합니다.  

ON 절은 2개 이상의 테이블을 결합할 때, 어떠한 조건으로 결합할지 알려줍니다.  

JOIN 절은 LEFT, INNER, RIGHT JOIN 3가지가 있으며, 다이어그램으로 표현하면 다음과 같습니다.  

<br/>

![MySQL_JOIN](/assets/img/DB_SQL/MY_SQL_JOIN.png)  

<br/>

## LEFT / RIGHT JOIN

LEFT JOIN 은 2개의 테이블에서 첫 번째(SQL 문에서 왼쪽에 있는) 테이블을 기준으로  

두 번째 테이블을 조합하는 방법입니다.  

<br/>

```sql
-- 1번 그림의 SQL
SELECT *
FROM Table_A as A
    LEFT JOIN Table_B as B
ON A.key = B.key;
-- WHERE B.key is null -- 2번 그림을 위한 WHERE절
```

<br/>

RIGHT JOIN 은 2개의 테이블에서 두 번째(SQL 문에서 오른쪽에 있는) 테이블을 기준으로  

첫 번째 테이블을 조합하는 방법입니다.  

<br/>

```sql
-- 4번 그림의 SQL
SELECT *
FROM Table_A as A
    RIGHT JOIN Table_B as B
ON A.key = B.key;
-- WHERE A.key is null -- 5번 그림을 위한 WHERE절
```

<br/>

## INNER JOIN

INNER JOIN 은 2개의 테이블에서 **ON 절 조건에 일치하는 결과**만 출력합니다.  

이때, INNER JOIN을 **생략하고 대신 콤마**를 사용할 수 있습니다.  

<br/>

```sql
-- 3번 그림의 SQL
SELECT *
FROM Table_A as A INNER JOIN Table_B as B
-- FROM Table_A as A, Table_B as B -- INNER JOIN 생략 가능
ON A.key = B.key;
```

<br/>

## JOIN 절 주의사항

JOIN 절에서 테이블 작성 순서에 따라 결과가 달라질 수 있습니다.  

따라서 어떤 테이블을 기준으로 이떤 테이블을 JOIN 해야 하는지 주의해서 사용해야 합니다.  

## JOIN 문제

[해커랭크 SQL 문제](https://www.hackerrank.com/challenges/the-company/problem?isFullScreen=true)  

<details>
<summary>문제 정답</summary>
<div markdown="1">

```sql
SELECT C.COMPANY_CODE, C.FOUNDER, 
    COUNT(DISTINCT LM.LEAD_MANAGER_CODE),
    COUNT(DISTINCT SM.SENIOR_MANAGER_CODE),
    COUNT(DISTINCT M.MANAGER_CODE),
    COUNT(DISTINCT E.EMPLOYEE_CODE)
FROM COMPANY as C 
    JOIN LEAD_MANAGER as LM ON C.COMPANY_CODE = LM.COMPANY_CODE
    JOIN SENIOR_MANAGER as SM ON LM.LEAD_MANAGER_CODE = SM.LEAD_MANAGER_CODE
    JOIN MANAGER as M ON SM.SENIOR_MANAGER_CODE = M.SENIOR_MANAGER_CODE
    JOIN EMPLOYEE as E ON M.MANAGER_CODE = E.MANAGER_CODE
GROUP BY C.COMPANY_CODE, C.FOUNDER
ORDER BY C.COMPANY_CODE;
```

</div>
</details