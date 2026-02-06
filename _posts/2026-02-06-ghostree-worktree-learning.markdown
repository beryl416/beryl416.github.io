---
layout: post
title: "[Ghostree] worktree 처음 써본 날 - 배운 것 정리"
date: 2026-02-06 20:30:00 +0900
categories: ai
---

오늘 처음으로 **Git worktree + Ghostree**를 제대로 써봤다. 결론부터 말하면, 브랜치를 여러 개 동시에 띄워놓고 작업하는 흐름이 훨씬 깔끔해졌다. 대신 처음엔 헷갈릴 포인트가 분명히 있어서, 내가 배운 걸 정리해둔다.

## 왜 worktree를 쓰는가

- 브랜치마다 **별도 폴더**가 생겨서 동시에 작업 가능
- `git stash` 없이도 병렬 작업 가능
- Ghostree로 **UI에서 생성/열기/삭제**가 쉬움
- Ghostty 창이 worktree마다 열리니 헷갈림이 줄어듦

## Ghostree로 worktree 만들기

1. Ghostree에서 레포 추가
2. `New worktree...` 클릭
3. Branch 이름 입력 (예: `feature-hero`)
4. Base는 `main` 선택
5. Create

주의: `main` worktree를 만들 때는 **Create branch 체크를 해제**해야 한다. 이미 존재하는 브랜치를 새로 만들려고 해서 에러가 뜬다.

## 내가 실제로 해본 흐름

정적 웹 실습을 만들고, worktree 2개에서 병렬로 기능을 추가했다.

- `feature-hero`: 히어로 섹션 추가
- `feature-theme`: 라이트 테마 토글 + 카드 섹션 추가

각 worktree에서 파일을 수정하고 커밋한 뒤, `main` worktree에서 병합했다.

```bash
git merge feature-hero
git merge feature-theme
```

## 병합할 때 생긴 문제: 충돌

두 브랜치가 **같은 파일(`index.html`, `styles.css`)의 같은 부분**을 건드리면 충돌이 난다. 이건 오류가 아니라 “직접 합쳐달라”는 신호다.

해결 방법:

1. 충돌 파일 열기
2. `<<<<<<<`, `=======`, `>>>>>>>` 구간을 정리
3. 원하는 최종 결과로 만들기
4. `git add` 후 커밋

## 머지 메시지 화면(에디터)이 뜰 때

두 번째 merge에서 **vim 화면**이 뜰 수 있다. 그건 머지 커밋 메시지를 쓰라는 화면이다.

- 저장 후 종료: `Esc` → `:wq` → Enter

에디터를 건너뛰고 싶으면:

```bash
git merge feature-theme -m "merge feature-theme"
```

## worktree 정리는 언제?

- 병합이 끝났고 더 쓸 일이 없으면 삭제하는 게 깔끔하다.
- Ghostree에서 worktree 선택 → Delete

실무에서는 **PR 머지 후 정리**가 기본 흐름이다.

## 내가 느낀 핵심 포인트

- worktree는 “동시에 작업하는 브랜치”에 가장 효과적
- Ghostree UI가 쉬워서 Git 초보도 접근 가능
- 충돌은 피할 수 없고, **빨리/명확하게 해결하는 습관**이 중요

## 보너스: 현재 worktree에서 Claude 바로 실행하기

Ghostree로 띄운 worktree 창에서 바로 Claude Code를 실행하려면, `~/.zshrc`에 간단히 함수 하나만 두면 된다.

```bash
cc() { claude --dangerously-skip-permissions; }
```

이후 `source ~/.zshrc` 하면 어느 worktree에서든 `cc` 한 번으로 실행된다.

---

정리하면, worktree는 **동시 작업**을 깔끔하게 해주는 도구이고, Ghostree는 그걸 **쉽게 쓰게 해주는 UI**다. 앞으로는 브랜치 여러 개를 오가야 할 때 worktree부터 만들게 될 것 같다.
