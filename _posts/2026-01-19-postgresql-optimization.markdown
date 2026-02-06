---
layout: post
title: "PostgreSQL 쿼리 최적화"
date: 2026-01-19
categories: database
---

PostgreSQL 쿼리를 최적화하는 팁입니다.

## EXPLAIN ANALYZE

```sql
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'test@test.com';
```

## 최적화 팁

- 적절한 인덱스 생성
- N+1 쿼리 주의
- 서브쿼리 대신 JOIN 활용
