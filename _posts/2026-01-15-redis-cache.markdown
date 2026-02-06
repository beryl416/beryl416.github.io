---
layout: post
title: "Redis 캐시 활용법"
date: 2026-01-15
categories: database
---

Redis를 캐시로 활용하는 방법입니다.

## 기본 명령

```
SET key value EX 3600
GET key
DEL key
```

## 캐시 전략

- Cache Aside
- Write Through
- Write Behind
