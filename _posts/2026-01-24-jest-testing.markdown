---
layout: post
title: "Jest로 테스트 코드 작성하기"
date: 2026-01-24
categories: testing
---

Jest로 테스트 코드를 작성하는 방법입니다.

## 기본 테스트

```javascript
test('adds 1 + 2', () => {
  expect(1 + 2).toBe(3);
});
```

## 주요 매처

- `toBe`, `toEqual`, `toBeTruthy`
