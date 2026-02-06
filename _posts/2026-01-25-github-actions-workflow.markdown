---
layout: post
title: "GitHub Actions 워크플로우"
date: 2026-01-25
categories: devops
---

GitHub Actions 워크플로우 작성법입니다.

## 기본 구조

```yaml
on: push
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm test
```
