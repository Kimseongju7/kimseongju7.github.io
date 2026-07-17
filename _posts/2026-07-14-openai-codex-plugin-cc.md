---
title: 'openai/codex-plugin-cc — Claude Code 안에서 Codex를 쓰는 공식 플러그인'
date: 2026-07-14 14:18:06 +0900
categories: [개발, 도구]
tags: [codex, claude-code, plugin, code-review, openai, ai-agent]
description: 'OpenAI가 공개한 codex-plugin-cc는 Claude Code 안에서 그대로 Codex를 불러 코드 리뷰를 받거나 작업을 위임할 수 있게 해주는 플러그인이다. 이미 Claude Code 워크플로를 쓰고 있는 사용자가 별도 전환 없이 Codex를 쓰기 시작할 수 있도록 만든 것이 목적이다.'
---
OpenAI가 공개한 **codex-plugin-cc**는 Claude Code 안에서 그대로 Codex를 불러 코드 리뷰를 받거나 작업을 위임할 수 있게 해주는 플러그인이다. 이미 Claude Code 워크플로를 쓰고 있는 사용자가 별도 전환 없이 Codex를 쓰기 시작할 수 있도록 만든 것이 목적이다.

## 제공 기능

- `/codex:review` — 일반 읽기 전용 Codex 코드 리뷰
- `/codex:adversarial-review` — 방향을 지정할 수 있는(steerable) 도전적 리뷰
- `/codex:rescue`, `/codex:transfer`, `/codex:status`, `/codex:result`, `/codex:cancel` — 작업 위임, 세션 핸드오프, 백그라운드 잡 관리

## 요구 사항과 설치

- **ChatGPT 구독(무료 포함) 또는 OpenAI API 키** 필요 — 사용량은 Codex 사용 한도에 합산된다.
- **Node.js 18.18 이상** 필요.

설치 순서는 다음과 같다.

```
/plugin marketplace add openai/codex-plugin-cc
/plugin install codex@openai-codex
/reload-plugins
/codex:setup
```

`/codex:setup`은 Codex가 준비됐는지 알려주고, Codex가 없고 npm이 있으면 대신 설치를 제안한다. 직접 설치하려면 `npm install -g @openai/codex`, 로그인이 안 돼 있으면 `!codex login`을 실행한다. 설치 후에는 슬래시 명령들과 `/agents` 안의 `codex:codex-rescue` 서브에이전트가 보인다.

## /codex:review — 일반 리뷰

현재 작업에 대해 Codex 안에서 `/review`를 직접 돌리는 것과 같은 품질의 리뷰를 수행한다. 커밋 안 된 변경 리뷰, 또는 `--base <ref>`로 `main` 같은 기준 브랜치 대비 브랜치 리뷰에 쓴다. `--wait`와 `--background`를 지원하며, 멀티 파일 변경 리뷰는 오래 걸릴 수 있어 백그라운드 실행이 권장된다. 읽기 전용이라 코드를 바꾸지 않으며, 커스텀 포커스 텍스트는 받지 않는다(그건 adversarial-review의 몫).

## /codex:adversarial-review — 도전적 리뷰

선택한 구현과 설계 자체에 의문을 제기하는 **조종 가능한** 리뷰다. 가정·트레이드오프·실패 모드를 압박 테스트하고, 더 안전하거나 단순한 다른 접근이 있었는지 따진다. `/codex:review`와 같은 대상 선택(`--base` 포함)을 쓰되, 플래그 뒤에 포커스 텍스트를 추가할 수 있다.

```
/codex:adversarial-review --base main challenge whether this was the right caching and retry design
/codex:adversarial-review --background look for race conditions and question the chosen approach
```

인증(auth), 데이터 손실, 롤백, 레이스 컨디션, 신뢰성 같은 특정 위험 영역을 집중 공략할 때 유용하다. 역시 읽기 전용이며 코드를 고치지는 않는다.

## /codex:rescue — 작업 위임

`codex:codex-rescue` 서브에이전트를 통해 작업을 Codex에 넘긴다. 버그 조사, 수정 시도, 이전 Codex 작업 이어가기, 더 작은 모델로 빠르고 저렴한 패스 돌리기에 쓴다. `--background`, `--wait`, `--resume`, `--fresh`를 지원하고, 둘 다 생략하면 해당 저장소의 최근 rescue 스레드를 이어갈지 제안할 수 있다.

```
/codex:rescue investigate why the tests started failing
/codex:rescue --model gpt-5.4-mini --effort medium investigate the flaky integration test
/codex:rescue --model spark fix the issue quickly
```

- `--model`/`--effort`를 안 주면 Codex가 자체 기본값을 고른다.
- `spark`라고 하면 `gpt-5.3-codex-spark`로 매핑된다.
- "Ask Codex to ..." 처럼 자연어로 위임을 요청해도 된다.

## /codex:transfer — 세션 핸드오프

현재 Claude Code 세션에서 영속적인 Codex 스레드를 만들고 `codex resume <session-id>` 명령을 출력한다. Claude Code에서 시작한 디버깅·구현 대화를 같은 맥락 그대로 Codex에서 이어가고 싶을 때 쓴다. 플러그인의 `SessionStart` 훅이 현재 트랜스크립트 경로를 자동 공급하고 `--source`는 수동 오버라이드다. 소스는 `~/.claude/projects` 아래여야 하고, 세션 임포트를 지원하지 않는 구버전 Codex는 업그레이드가 필요하다.

## 백그라운드 잡 관리

- `/codex:status` — 현재 저장소의 실행 중/최근 Codex 잡을 보여준다.
- `/codex:result` — 완료된 잡의 최종 출력을 보여주고, 가능하면 Codex 세션 ID도 함께 줘서 `codex resume`으로 그 실행을 다시 열 수 있다.
- `/codex:cancel` — 진행 중인 백그라운드 잡을 취소한다.

## 리뷰 게이트 (선택 기능)

```
/codex:setup --enable-review-gate
/codex:setup --disable-review-gate
```

리뷰 게이트를 켜면 `Stop` 훅으로 Claude의 응답에 기반한 표적 Codex 리뷰가 돌고, 문제가 발견되면 종료가 차단되어 Claude가 먼저 그 문제를 처리하게 된다. 단, **Claude/Codex 간 긴 루프가 생겨 사용 한도를 빠르게 소진할 수 있으므로** 세션을 적극적으로 지켜볼 때만 켜라는 경고가 붙어 있다.

## Codex 연동 방식과 설정

플러그인은 Codex app server를 감싸며, 환경에 설치된 전역 `codex` 바이너리와 동일한 설정을 그대로 쓴다. 기본 모델이나 reasoning effort를 바꾸려면 `~/.codex/config.toml`(사용자 레벨) 또는 프로젝트 루트의 `.codex/config.toml`(프로젝트가 신뢰된 경우에만 로드)에 지정한다.

```
model = "gpt-5.4-mini"
model_reasoning_effort = "high"
```

FAQ 요점: 별도 Codex 계정은 필요 없고 로컬 Codex CLI 인증을 그대로 쓴다. 별도 런타임도 아니어서 같은 Codex 설치·인증 상태·저장소 체크아웃을 사용한다. 기존 API 키나 `openai_base_url` 커스텀 엔드포인트 설정도 그대로 적용된다.

## 관련 노트

- [gpt-5-6-sol-in-claude-code](/posts/gpt-5-6-sol-in-claude-code/) — 이 플러그인(루트 A)으로 Claude Code 안에서 GPT-5.6 Sol을 리뷰어로 붙이는 실전 절차
- [harness-engineering-codex](/posts/harness-engineering-codex/) — Codex를 하네스 관점에서 다루는 배경
- [codex-changelog](/posts/codex-changelog/) — Codex CLI 릴리스·기능 변화 추적
- [codex-top-6-skills](/posts/codex-top-6-skills/) — Codex에서 함께 쓰기 좋은 스킬 모음

> 원문: [openai/codex-plugin-cc: Use Codex from Claude Code to review code or delegate tasks.](https://github.com/openai/codex-plugin-cc)
> 원본 클립: 2026-07-14-openaicodex-plugin-cc Use Codex from Claude Code to review code or delegate tasks.
