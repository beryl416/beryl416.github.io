---
layout: post
title: "SSH 키 설정과 관리"
date: 2026-01-28
categories: tools
---

SSH 키를 설정하고 관리하는 방법입니다.

## 키 생성

```bash
ssh-keygen -t ed25519 -C "email@example.com"
```

## config 파일

```
Host github
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519
```
