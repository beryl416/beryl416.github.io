---
layout: post
title: "Nginx 리버스 프록시 설정"
date: 2026-01-23
categories: devops
---

Nginx를 리버스 프록시로 설정하는 방법입니다.

## 기본 설정

```nginx
server {
    listen 80;
    location / {
        proxy_pass http://localhost:3000;
    }
}
```
