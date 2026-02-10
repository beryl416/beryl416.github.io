---
layout: post
title: "[GitHub Actions] CI 알아보기"
date: 2026-02-10 18:10:00 +0900
categories: ai
---

코드를 만들다 보면 이런 일이 자주 생깁니다.

- push 하면 GitHub Actions가 자동으로 테스트를 돌립니다.
- 코드 수정 후 그냥 push하면 됩니다.
- GitHub Actions가 자동으로 다음을 실행합니다.
  - 테스트
  - 빌드
  - 린트 체크

이게 바로 **CI (Continuous Integration)** 입니다.

## 핵심 포인트

- `push` 또는 `PR` 같은 이벤트를 트리거로 실행
- 자동으로 실행
- 같은 환경에서 실행
- 항상 같은 검증을 실행
