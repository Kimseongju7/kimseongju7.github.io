---
title: "[복붙 OK] Claude Code 안에서 GPT-5.6 Sol을 돌리는 방법"
date: 2026-07-14 14:04:08 +0900
categories: [개발, 도구]
tags: [claude-code, gpt, codex, proxy, harness, ai, llm]
slug: gpt-5-6-sol-in-claude-code
publish: true
---
ChatGPT 유료 플랜(Codex 권한)만 있으면 추가 API 과금 없이 OpenAI의 최신 코딩 모델 GPT-5.6 Sol을 Claude Code 안에서 돌릴 수 있다는 글. 해외에서는 이미 "gpt-5.6-sol은 Codex 안에서보다 Claude Code 안에서가 명백히 좋다"는 이야기가 돌고 있는데, 그 차이를 만든 것은 모델이 아니라 하니스(토대)라는 것이 본론이다.

## 왜 본가(Codex)보다 Claude Code에서 더 잘 도는가

- 성능을 가른 것은 모델 자체가 아니라 하니스 쪽이었다. 모델이 엔진이고 Claude Code가 차체 — 같은 엔진이라도 차체 설계가 좋으면 더 빨리 달린다.
- Claude Code는 도구 호출 방식, 자식 에이전트 제어, 사고 깊이(effort) 조절 같은 "모델 바깥"의 만듦새가 두텁다.
- T3 스택 제작자(개발자 커뮤니티에서 수십만 명이 동향을 좇는 실무자)도 "subagent의 effort를 직접 고정할 수 있는 점은 Claude Code가 훨씬 앞서 있다"고 말했다.
- 속이 같다면 바깥이 정성스러운 쪽이 실력을 낸다 — 그게 전부다.

## 전제와 작동 원리

- ChatGPT 유료 플랜(Codex 쿼터)에 과금 중이면 그 권리만으로 GPT-5.6 Sol을 Claude Code에서 쓸 수 있고, 별도 API 과금은 불필요하다.
- 구조는 "중간에 통역을 한 명 세우는" 그림이다. Claude Code는 Anthropic 형식만 말하고 Sol은 OpenAI 형식으로만 답하므로, 직접 대화가 안 된다.
- 그래서 **CLIProxyAPI**라는 통역(자기 PC에서 도는 프록시)을 끼우고, ChatGPT 유료 로그인을 그대로 통역의 신분증으로 쓴다. 그래서 추가 과금이 없다.

## 루트 A — 공식 플러그인판 (안전, Claude와 병용)

```
/plugin marketplace add openai/codex-plugin-cc
/plugin install codex@openai-codex
/reload-plugins
/codex:setup
```

- 이걸로 `/codex:review`, `/codex:adversarial-review`, `/codex:rescue`를 쓸 수 있다.
- Claude가 쓰고, GPT가 비평하고, 내가 출하한다. Claude를 주역으로 남긴 채 GPT를 리뷰 담당으로 추가하는 방식.
- 처음 시도한다면 이쪽이 안전하다.

## 루트 B — 프록시판 (Sol을 메인으로)

1. **Step 1**: CLIProxyAPI 설치 (설정 파일 `config.yaml`에 포트 번호 8317과 직접 정한 API 키만 적으면 됨)
2. **Step 2**: 접속 (ChatGPT와 Claude 인증을 통과시켜 Claude Code에 연결)
3. **Step 3**: 아래 에일리어스를 정의하고 `claudex`를 즐긴다

### 루트 B-1 — Sol 상시 메인판

```
alias claudex='ANTHROPIC_BASE_URL=http://127.0.0.1:8317 \
ANTHROPIC_AUTH_TOKEN=자기키 \
CLAUDE_CODE_SUBAGENT_MODEL=gpt-5.6-sol \
CLAUDE_CODE_ALWAYS_ENABLE_EFFORT=1 \
CLAUDE_CODE_MAX_TOOL_USE_CONCURRENCY=3 \
ENABLE_TOOL_SEARCH=false \
claude --model gpt-5.6-sol'
```

- `claudex`를 치는 순간부터 Sol이 메인. subagent 모델과 effort(사고 깊이)도 환경변수로 고정할 수 있다.
- Sol을 본격적으로 파고들 사람용.

### 루트 B-2 — 모델 전환식 (평소엔 Claude, 필요할 때만 Sol)

```
alias claudex='ANTHROPIC_BASE_URL=http://127.0.0.1:8317 \
ANTHROPIC_AUTH_TOKEN=자기키 \
claude'
```

- 평소엔 늘 쓰던 Claude 그대로, `/model gpt-5.6-sol`을 쳤을 때만 Sol로 전환. 되돌리는 것도 `/model` 한 방.
- B-1과의 차이는 마지막 `--model gpt-5.6-sol`을 붙이느냐 떼느냐뿐이다.
- 글쓴이는 이 방식을 쓴다 — Claude의 지휘를 남긴 채, 세컨드 오피니언이나 무거운 구현 일회성 의뢰만 Sol에 던질 수 있어서.

### 세 루트 비교

- **루트 A (GPT를 리뷰 담당으로)**: 평소처럼 `claude`로 기동, `/codex:review`로 GPT가 비평. 필요한 것은 플러그인 + Codex CLI. → 먼저 안전하게 시험해 보고 싶은 사람용
- **루트 B-1 (Sol 상시 메인)**: `claudex` 기동 즉시 모든 응답이 Sol. 필요한 것은 CLIProxyAPI + 인증 2개. → Sol을 본격적으로 쓸 사람용
- **루트 B-2 (평소 Claude, 가끔 Sol)**: `claudex` 기동, `/model gpt-5.6-sol` 때만 Sol. 필요한 것은 B-1과 동일(`--model`만 제거). → Claude의 지휘를 남기고 싶은 사람용

Claude Code에 이 글을 던지면 절차 자체는 몇 분이면 끝난다.

## 결국 무슨 이야기인가

- "AI는 어느 모델을 쓰면 되나?"의 답은 더 이상 하나를 고르는 것이 아니다. 그릇(하니스)을 먼저 정하고, 그 안에 누구를 어떻게 배치할지의 문제다.
- Claude Code를 토대로 Fable을 지휘역, Sol·Grok·Sonnet을 실행 부대로 편성한다. 새 모델이 나오면 전부 갈아타는 게 아니라 실행 부대에 신전력 한 장을 더한다.
- 이번 Sol 소동도 결국 이 편성 이야기의 일부였다.

## 글쓴이의 주장: AI 시대를 어떻게 살아남을 것인가

- 진짜 의미는 "개인이 'AI 팀'을 가질 수 있게 됐다"는 것. 설계하는 사람, 구현하는 사람, 리뷰하는 사람 — 얼마 전까지 회사가 여러 명을 고용해서 하던 일을 한순간에 AI 팀으로 편성할 수 있다.
- 글쓴이는 외주가 "1개월 걸립니다"라던 작업이 Claude Code로 3시간에 끝나는 것을 여러 번 봤다고 한다. 실행 비용이 거의 사라져 아이디어가 그대로 사업이 된다.
- 의미 있는 업데이트가 2~4개월마다 오는 지금은 아직 "초기" — 인터넷으로 치면 1995년쯤의 공기이며, 지금 시작하는 사람은 전원 "빠른 쪽"이다.
- 도구 사용법만 배우고 끝나는 사람이 9할이지만, 차이는 "어떻게 쓰느냐"보다 "어디에 쓰느냐"에서 압도적으로 벌어진다는 주장. (글 말미는 글쓴이의 오픈채팅/팔로우 홍보로 이어진다.)

## 관련 노트

- [[openai-codex-plugin-cc]] — 루트 A에서 쓰는 공식 codex-plugin-cc의 명령·설치 상세
- [[claude-code-model-and-effort]] — subagent 모델·effort(사고 깊이)를 고정한다는 이 글의 핵심 축
- [[prompting-claude-fable-5]] — Fable을 지휘역으로 두는 편성 이야기와 이어지는 Fable 프롬프팅

> 원문: [【コピペでOK】ClaudeCodeの中でGPT-5.6 Solを動かす方法](https://x.com/chizevsm5/status/2076596525262876890)
> 원본 클립: [[2026-07-14-【コピペでOK】ClaudeCodeの中でGPT-5.6 Solを動かす方法]]
