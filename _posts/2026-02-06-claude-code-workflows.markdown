---
layout: post
title: "Claude Code 일반적인 워크플로우"
date: 2026-02-06 18:00:00 +0900
categories: ai
---

[첫 번째 글](/ai/2026/02/06/claude-code-for-kids.html)에서 Claude Code가 어떻게 작동하는지, [두 번째 글](/ai/2026/02/06/claude-code-extensions.html)에서 확장 기능을 알아봤어. 이번에는 **실제로 매일 쓰는 워크플로우**를 알아볼 거야.

코드 탐색, 버그 수정, 리팩토링, 테스트 작성, PR 만들기까지 — Claude Code로 할 수 있는 일상적인 작업들을 단계별로 설명해줄게.

## 새로운 코드베이스 이해하기

### 프로젝트 전체를 빠르게 파악하기

새로운 프로젝트에 처음 참여했을 때, 구조를 빠르게 이해하고 싶잖아? 이럴 때 Claude에게 물어보면 돼.

```
cd /path/to/project
claude
```

그리고 이렇게 질문해:

```
> 이 코드베이스 전체 개요를 알려줘
> 여기서 사용하는 주요 아키텍처 패턴을 설명해줘
> 핵심 데이터 모델이 뭐야?
> 인증은 어떻게 처리돼?
```

넓은 질문으로 시작해서 점점 좁혀나가는 게 좋아. "이 프로젝트에서 쓰는 용어 정리해줘"라고 하면 용어집도 만들어줘.

### 관련 코드 찾기

특정 기능과 관련된 코드를 찾아야 할 때:

```
> 사용자 인증을 처리하는 파일들을 찾아줘
> 이 인증 파일들이 어떻게 함께 작동해?
> 로그인 과정을 프론트엔드부터 데이터베이스까지 추적해줘
```

찾고 싶은 것을 **구체적으로** 설명하고, 프로젝트에서 쓰는 용어를 그대로 쓰면 더 잘 찾아줘.

## 버그 수정하기

에러가 나타났을 때, Claude한테 그대로 보여주면 돼:

```
> npm test를 실행하면 에러가 나
```

Claude가 원인을 분석하고 수정 방법을 제안해줘:

```
> user.ts에 있는 @ts-ignore를 고칠 방법 몇 가지 제안해줘
```

마음에 드는 방법을 선택하면:

```
> user.ts에 네가 제안한 null 체크를 추가해줘
```

스택 트레이스(에러 메시지 전체)를 Claude에게 보여주고, 에러가 항상 나는지 가끔 나는지 알려주면 더 정확하게 고쳐줘.

## 코드 리팩토링

오래된 코드를 최신 방식으로 바꾸고 싶을 때:

```
> 코드베이스에서 더 이상 사용되지 않는 API 사용을 찾아줘
> utils.js를 최신 JavaScript 기능으로 리팩토링하는 방법을 제안해줘
> utils.js를 같은 동작을 유지하면서 ES2024 기능으로 리팩토링해줘
> 리팩토링한 코드의 테스트를 실행해줘
```

한 번에 너무 많이 바꾸지 말고, **작은 단위로 나눠서** 리팩토링하고 매번 테스트를 돌리는 게 안전해.

## Subagents 활용하기

특정 작업을 더 잘 처리하기 위해 **전문 도우미(Subagent)**를 쓸 수 있어.

```
> /agents
```

이 명령으로 사용 가능한 Subagent 목록을 보거나 새로 만들 수 있어.

Claude는 알아서 적절한 Subagent에게 일을 맡기기도 해:

```
> 내 최근 코드 변경사항에서 보안 문제를 검토해줘
> 모든 테스트를 실행하고 실패하는 것들을 고쳐줘
```

직접 지정할 수도 있어:

```
> code-reviewer subagent를 사용해서 auth 모듈을 검토해줘
```

나만의 Subagent도 만들 수 있어. `/agents`를 실행하고 "Create New subagent"를 선택하면, 이름, 설명, 사용할 도구, 역할을 정의할 수 있어. `.claude/agents/` 폴더에 저장하면 팀원들과 공유도 돼.

## Plan Mode — 안전하게 분석하기

Plan Mode는 Claude가 **읽기만 하고 아무것도 안 고치는** 모드야. 코드를 살펴보고 계획만 세워줘.

### 언제 쓸까?

- 여러 파일을 수정해야 하는 큰 작업을 **계획**할 때
- 코드를 바꾸기 전에 **먼저 조사**하고 싶을 때
- Claude와 **방향을 맞춰가면서** 작업하고 싶을 때

### 사용 방법

**세션 중에 전환**: `Shift+Tab`을 누르면 모드가 바뀌어. 두 번 누르면 Plan Mode(`⏸ plan mode on`)로 들어가.

**처음부터 Plan Mode로 시작**:

```bash
claude --permission-mode plan
```

**바로 질문하기**:

```bash
claude --permission-mode plan -p "인증 시스템을 분석하고 개선점을 제안해줘"
```

### 예시

```
> 인증 시스템을 OAuth2로 리팩토링해야 해.
> 상세한 마이그레이션 계획을 세워줘.
```

Claude가 현재 코드를 분석하고 계획을 세워줘. 후속 질문으로 다듬을 수 있어:

```
> 하위 호환성은 어떻게 해?
> 데이터베이스 마이그레이션은 어떻게 처리해야 해?
```

`Ctrl+G`를 누르면 계획을 텍스트 편집기에서 직접 수정할 수도 있어.

Plan Mode를 기본으로 쓰고 싶으면:

```json
// .claude/settings.json
{
  "permissions": {
    "defaultMode": "plan"
  }
}
```

## 테스트 작성하기

테스트가 없는 코드에 테스트를 추가해야 할 때:

```
> NotificationsService.swift에서 테스트가 없는 함수들을 찾아줘
> 알림 서비스에 테스트를 추가해줘
> 알림 서비스의 엣지 케이스에 대한 테스트를 추가해줘
> 새 테스트를 실행하고 실패하는 것들을 고쳐줘
```

Claude는 프로젝트에 이미 있는 테스트 스타일을 보고 **같은 패턴**으로 테스트를 만들어줘. 테스트 프레임워크, 어설션 방식, 파일 구조까지 맞춰서.

어떤 동작을 테스트하고 싶은지 구체적으로 말해주면 좋아. 그리고 "놓친 엣지 케이스 있어?"라고 물어보면, Claude가 에러 조건, 경계값, 예상치 못한 입력에 대한 테스트도 제안해줘.

## Pull Request 만들기

변경사항을 PR로 만들 때, 가장 간단한 방법:

```
> /commit-push-pr
```

이 한 명령으로 커밋, 푸시, PR 생성까지 한 번에 돼. Slack MCP 서버를 연결해두고 CLAUDE.md에 채널을 지정하면, PR URL을 자동으로 Slack에도 올려줘.

단계별로 하고 싶으면:

```
> 내가 인증 모듈에 만든 변경사항을 요약해줘
> PR을 만들어줘
> 보안 개선에 대한 더 많은 맥락으로 PR 설명을 보강해줘
```

`gh pr create`로 PR을 만들면 세션이 자동으로 그 PR에 연결돼. 나중에 `claude --from-pr 123`으로 이어서 작업할 수 있어.

## 문서 작성하기

코드에 문서를 추가해야 할 때:

```
> auth 모듈에서 JSDoc 주석이 없는 함수들을 찾아줘
> auth.js에서 문서화되지 않은 함수들에 JSDoc 주석을 추가해줘
> 생성된 문서에 더 많은 맥락과 예제를 추가해서 개선해줘
> 문서가 우리 프로젝트 기준에 맞는지 확인해줘
```

원하는 문서 스타일(JSDoc, docstrings 등)을 지정하고, 예제를 포함해달라고 하면 더 좋은 문서가 나와.

## 이미지로 작업하기

Claude는 이미지도 이해할 수 있어. 에러 스크린샷, UI 디자인, 다이어그램 등을 보여주면 분석해줘.

**이미지 넣는 방법:**
1. Claude Code 창에 이미지를 **드래그 앤 드롭**
2. 이미지를 복사한 후 `Ctrl+V`로 붙여넣기
3. 경로를 직접 알려주기: "Analyze this image: /path/to/image.png"

```
> 에러 스크린샷이야. 원인이 뭐야?
> 현재 데이터베이스 스키마야. 어떻게 수정해야 해?
> 이 디자인 목업에 맞는 CSS를 만들어줘
```

텍스트로 설명하기 어려운 것(에러 화면, UI 디자인, 다이어그램)은 이미지를 보여주는 게 훨씬 빨라.

## 파일 참조하기 — @ 사용

`@`를 쓰면 파일이나 디렉토리를 빠르게 참조할 수 있어:

```
> @src/utils/auth.js의 로직을 설명해줘
> @src/components의 구조가 어떻게 돼?
```

MCP 리소스도 참조 가능해:

```
> @github:repos/owner/repo/issues에서 데이터를 보여줘
```

한 메시지에서 여러 파일을 참조할 수도 있어: `@file1.js and @file2.js`

## 확장된 사고 (Extended Thinking)

Extended thinking은 기본적으로 켜져 있어. Claude가 답하기 전에 **복잡한 문제를 단계별로 생각**할 시간을 주는 거야.

복잡한 아키텍처 결정, 어려운 버그, 다단계 구현 계획을 세울 때 특히 유용해.

### 설정 방법

| 설정 | 방법 | 설명 |
|------|------|------|
| Effort level | `/model`에서 조정 | low, medium, high 중 선택 |
| 켜기/끄기 | `Option+T` (Mac) / `Alt+T` (Windows) | 현재 세션에서 토글 |
| 전역 기본값 | `/config` | 모든 프로젝트 기본값 설정 |
| 토큰 제한 | `MAX_THINKING_TOKENS` 환경변수 | 생각하는 양을 제한 |

Claude가 생각하는 과정을 보고 싶으면 `Ctrl+O`로 verbose mode를 켜면 회색 이탤릭 텍스트로 내부 추론이 보여.

**Opus 4.6**은 적응형 추론을 써서 effort level에 따라 생각하는 양을 자동으로 조절해. 다른 모델은 최대 31,999 토큰의 고정 예산을 써.

## 이전 대화 재개하기

Claude와 했던 대화를 다시 이어갈 수 있어:

- `claude --continue` — 가장 최근 대화를 이어가기
- `claude --resume` — 대화 선택기 열기
- `claude --from-pr 123` — 특정 PR에 연결된 세션 재개

### 세션에 이름 붙이기

```
> /rename auth-refactor
```

나중에 이름으로 바로 찾을 수 있어:

```bash
claude --resume auth-refactor
```

### 세션 선택기 단축키

`/resume`을 실행하면 대화형 선택기가 열려:

| 단축키 | 기능 |
|--------|------|
| `↑`/`↓` | 세션 이동 |
| `Enter` | 세션 선택 |
| `P` | 미리보기 |
| `R` | 이름 바꾸기 |
| `/` | 검색 |
| `B` | 현재 브랜치 필터 |
| `Esc` | 종료 |

별개의 작업을 시작할 때 `/rename`으로 이름을 붙여두면 나중에 찾기 훨씬 쉬워.

## Git Worktrees로 병렬 작업하기

여러 작업을 **동시에** 하고 싶을 때, Git worktrees를 쓰면 같은 프로젝트에서 **여러 브랜치를 별도 폴더**로 체크아웃할 수 있어.

```bash
# 새 worktree 만들기
git worktree add ../project-feature-a -b feature-a
git worktree add ../project-bugfix bugfix-123

# 각 worktree에서 Claude 실행
cd ../project-feature-a && claude
cd ../project-bugfix && claude

# worktree 목록 보기
git worktree list

# 다 쓰면 제거
git worktree remove ../project-feature-a
```

각 worktree는 **완전히 독립된 파일**을 가지고 있어서, 한 쪽에서 바꿔도 다른 쪽에 영향 없어. 하지만 Git 기록과 원격 연결은 공유해.

## Claude를 Unix 도구처럼 쓰기

### 빌드 스크립트에 넣기

Claude를 린터처럼 쓸 수 있어:

```json
// package.json
{
  "scripts": {
    "lint:claude": "claude -p 'you are a linter. please look at the changes vs. main and report any issues related to typos. report the filename and line number on one line, and a description of the issue on the second line. do not return any other text.'"
  }
}
```

### 파이프로 연결하기

데이터를 Claude에게 보내고 결과를 파일로 받기:

```bash
cat build-error.txt | claude -p 'concisely explain the root cause of this build error' > output.txt
```

### 출력 형식 지정하기

```bash
# 일반 텍스트 (기본값)
cat data.txt | claude -p 'summarize this data' --output-format text > summary.txt

# JSON 형식 (메타데이터 포함)
cat code.py | claude -p 'analyze this code for bugs' --output-format json > analysis.json

# 스트리밍 JSON (실시간)
cat log.txt | claude -p 'parse this log file for errors' --output-format stream-json
```

## Claude에게 Claude에 대해 물어보기

Claude는 자기 자신의 문서에 접근할 수 있어서, Claude Code의 기능이나 사용법에 대해 물어보면 바로 답해줘:

```
> Claude Code로 pull request 만들 수 있어?
> Claude Code가 권한을 어떻게 처리해?
> 사용 가능한 skill이 뭐가 있어?
> Claude Code에서 MCP를 어떻게 써?
> Claude Code의 제한 사항이 뭐야?
```

항상 최신 문서를 기반으로 답해줘. 구체적으로 물어볼수록 자세한 답을 줘.

## 정리하면

| 워크플로우 | 핵심 |
|-----------|------|
| 코드베이스 이해 | 넓은 질문 → 좁혀가기 |
| 버그 수정 | 에러 메시지 + 재현 단계 보여주기 |
| 리팩토링 | 작은 단위로 나눠서, 매번 테스트 |
| Subagents | `/agents`로 전문 도우미 활용 |
| Plan Mode | `Shift+Tab` 두 번, 읽기만 하고 계획 세우기 |
| 테스트 | 기존 패턴 따라 작성, 엣지 케이스 추가 |
| PR 만들기 | `/commit-push-pr` 한 방에 |
| 이미지 | 드래그 앤 드롭 or `Ctrl+V` |
| 파일 참조 | `@파일경로`로 빠르게 |
| Extended Thinking | 복잡한 문제에서 깊이 생각 |
| 세션 재개 | `--continue`, `--resume`, `/rename` |
| 병렬 작업 | Git worktrees로 격리 |
| Unix 도구 | 파이프 연결, 빌드 스크립트 통합 |

이 워크플로우들을 하나씩 써보면서 익숙해지면 돼. 전부 다 외울 필요 없어 — 필요할 때 돌아와서 참고하면 되니까!
