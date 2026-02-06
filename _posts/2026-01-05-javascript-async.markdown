---
layout: post
title: "JavaScript 비동기 처리"
date: 2026-01-05
categories: javascript
---

JavaScript의 비동기 처리 방식을 정리합니다.

## 콜백에서 Promise, async/await까지

```javascript
async function fetchData() {
  const res = await fetch(url);
  return res.json();
}
```
