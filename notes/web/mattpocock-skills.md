---
title: "mattpocock/skills — 진짜 엔지니어를 위한 에이전트 스킬 모음"
date: 2026-07-14 14:35:33 +0900
categories: [개발, 도구]
tags: [claude-code, skills, ai-agent, tdd, code-review, matt-pocock, ddd]
slug: mattpocock-skills
publish: true
---
Matt Pocock이 자신의 `.claude` 디렉터리에서 매일 실제 엔지니어링에 쓰는 에이전트 스킬들을 공개한 저장소다. GSD·[[bmad-method|BMAD]]·Spec-Kit처럼 프로세스를 통째로 소유하는 접근과 달리, 작고 수정하기 쉽고 조합 가능한 스킬로 제어권을 사용자에게 남기는 것이 철학이다.

## 철학

- 실제 애플리케이션 개발은 어렵다. GSD, BMAD, Spec-Kit 같은 접근은 프로세스를 대신 소유해 주지만, 그만큼 제어권을 빼앗고 프로세스상의 버그를 고치기 어렵게 만든다.
- 이 스킬들은 작고, 적응시키기 쉽고, 조합 가능하게 설계됐다. 어떤 모델과도 동작하며, 수십 년의 엔지니어링 경험에 기반한다. 마음껏 고쳐서 자기 것으로 만들라는 취지.

## 설치 방법 (두 가지 철학)

1. **skills.sh 설치** — 스킬을 프로젝트에 복사해 직접 수정하며 자기 것으로 만드는 방식:
```
npx skills@latest add mattpocock/skills
```
설치 시 `/setup-matt-pocock-skills`를 반드시 선택하고, 에이전트에서 실행하면 이슈 트래커(GitHub·Linear·로컬 파일), 트리아지 라벨, 문서 저장 위치를 물어본 뒤 설정이 끝난다.

2. **Claude Code 플러그인** — 직접 관리하지 않는 읽기 전용 번들로 설치해 새 버전을 구독하는 방식:
```
/plugin marketplace add mattpocock/skills
/plugin install mattpocock-skills@mattpocock
```
설치 후 리포지터리마다 `/setup-matt-pocock-skills`를 한 번 실행한다.

- Codex 등 다른 에이전트도 skills.sh 인스톨러가 Agent-Skills 표준 하네스에 설치해 준다. 네이티브 Codex 플러그인은 로드맵에 있다.

## 이 스킬들이 해결하는 4가지 실패 모드

### 1. 에이전트가 내가 원한 걸 만들지 않았다

- 소프트웨어 개발의 가장 흔한 실패는 어긋남(misalignment)이다. AI 시대에도 마찬가지로, 나와 에이전트 사이에 소통 간극이 있다.
- 해결책은 **그릴링 세션(grilling session)** — 에이전트가 무엇을 만들지에 대해 나에게 상세한 질문을 퍼붓게 하는 것.
- `/grill-me`(비코드 용도), `/grill-with-docs`(같은 것 + 문서화 기능 추가)가 가장 인기 있는 스킬이며, 변경을 시작하기 전 매번 쓰라고 권한다.

### 2. 에이전트가 너무 장황하다

- 에이전트는 보통 프로젝트에 투입되어 전문 용어(jargon)를 스스로 파악해야 하므로, 한 단어면 될 것을 20단어로 말한다.
- 해결책은 **공유 언어(shared language)** — 프로젝트 용어를 해독하게 돕는 문서. 예: "코스 섹션 안의 레슨이 파일 시스템에 자리를 얻어 '실체화'될 때 문제가 있다" 대신 "materialization cascade에 문제가 있다".
- `/grill-with-docs`에 내장되어 있으며, 공유 언어를 만들고 설명하기 어려운 결정을 ADR로 문서화한다.
- 부수 효과: 변수·함수·파일 명명이 일관되고, 에이전트가 코드베이스를 탐색하기 쉬워지며, 더 간결한 언어 덕에 생각(thinking)에 쓰는 토큰도 줄어든다.

### 3. 코드가 동작하지 않는다

- 정렬이 되어도 결과물이 엉망이라면 피드백 루프를 봐야 한다. 코드가 실제로 어떻게 도는지에 대한 피드백이 없으면 에이전트는 눈감고 비행하는 셈이다.
- 정적 타입, 브라우저 접근, 자동화 테스트라는 통상적인 피드백 루프가 필요하다.
- `/tdd` 스킬은 실패하는 테스트를 먼저 쓰고 고치는 red-green-refactor 루프를 장려하고, 좋은 테스트와 나쁜 테스트에 대한 가이드를 제공한다.
- `/diagnosing-bugs` 스킬은 모범 디버깅 관행을 단순한 루프로 감쌌다.

### 4. 진흙 덩어리(ball of mud)를 만들어 버렸다

- 에이전트는 코딩을 급가속시키는 만큼 소프트웨어 엔트로피도 가속시켜, 코드베이스가 전례 없는 속도로 복잡해진다.
- 해결책은 코드 설계에 신경 쓰는 것. `/to-spec`은 스펙 작성 전에 어떤 모듈을 건드리는지 질문하고, `/improve-codebase-architecture`는 진흙 덩어리가 된 코드베이스를 구조하는 스킬로 며칠에 한 번씩 돌리기를 권한다.

## 스킬 레퍼런스

호출 주체 기준으로 나뉜다. **사용자 호출(user-invoked)** 스킬은 직접 타이핑해야만 실행되며 오케스트레이션 담당, **모델 호출(model-invoked)** 스킬은 에이전트가 작업에 맞으면 자동으로 꺼내 쓸 수 있으며 재사용 가능한 규율을 담는다. 사용자 호출 스킬은 모델 호출 스킬을 부를 수 있지만 다른 사용자 호출 스킬은 부르지 않는다.

### 엔지니어링 (사용자 호출)

- **ask-matt** — 상황에 맞는 스킬·플로를 안내하는 라우터
- **grill-with-docs** — 그릴링 + 도메인 모델 구축, CONTEXT.md·ADR 인라인 갱신
- **triage** — 트리아지 역할 상태 머신으로 이슈 처리
- **improve-codebase-architecture** — 코드베이스의 심화(deepening) 기회를 스캔해 HTML 리포트로 제시
- **setup-matt-pocock-skills** — 리포지터리별 초기 설정
- **to-spec** — 현재 대화를 스펙으로 만들어 이슈 트래커에 게시
- **to-tickets** — 계획·스펙·대화를 차단 관계가 명시된 트레이서 불릿 티켓들로 분해
- **implement** — 스펙/티켓 기반 구현, 합의된 지점에서 `/tdd`를 구동하고 커밋 전 `/code-review`로 마무리
- **wayfinder** — 한 세션에 담기지 않는 큰 작업을 조사 티켓 맵으로 계획

### 엔지니어링 (모델 호출)

- **prototype** — 설계 질문에 답하기 위한 일회용 프로토타입 제작
- **diagnosing-bugs** — 재현 → 최소화 → 가설 → 계측 → 수정 → 회귀 테스트의 규율 있는 진단 루프
- **research** — 신뢰도 높은 1차 소스를 조사해 인용 포함 Markdown으로 기록 (백그라운드 에이전트)
- **tdd** — red-green-refactor 루프, 수직 슬라이스 단위 개발
- **domain-modeling** — 용어를 용어집과 대조하고 엣지 케이스로 스트레스 테스트하며 도메인 모델을 다듬기
- **codebase-design** — 깊은 모듈(작은 인터페이스 뒤의 많은 동작) 설계를 위한 공유 규율·어휘
- **code-review** — 표준(코딩 표준 + Fowler 스멜) / 스펙(원 이슈·PRD 충실 구현) 두 축을 병렬 서브에이전트로 검토
- **resolving-merge-conflicts** — 머지·리베이스 충돌을 헝크 단위로 의도 기반 해소, 절대 `--abort` 하지 않음

### 생산성

- **grill-me** (사용자 호출) — 결정 트리의 모든 가지가 해소될 때까지 집요하게 인터뷰 받기
- **handoff** (사용자 호출) — 현재 대화를 다른 에이전트가 이어받을 인수인계 문서로 압축
- **teach** (사용자 호출) — 현재 디렉터리를 상태 유지 학습 작업 공간 삼아 여러 세션에 걸쳐 개념을 가르침
- **writing-great-skills** (사용자 호출) — 예측 가능한 스킬을 쓰기 위한 어휘와 원칙 레퍼런스
- **grilling** (모델 호출) — grill-me와 grill-with-docs 뒤에 있는 재사용 인터뷰 루프

## 관련 노트

- [[midudev-autoskills]] — 스택을 감지해 스킬을 자동 설치하는 도구
- [[codex-top-6-skills]] — Codex 에이전트를 확장하는 대표 스킬 6선
- [[alirezarezvani-claude-skills]] — 또 다른 Claude 스킬 모음
- [[ouroboros-agent-os]] — 그릴링/인터뷰로 스펙을 결정화하는 명세 우선 접근
- [[bmad-method|BMAD-METHOD]] — 프로세스를 통째로 소유하는 대비 접근으로 본문에서 비교되는 프레임워크

> 원문: [mattpocock/skills: Skills for Real Engineers. Straight from my .claude directory.](https://github.com/mattpocock/skills)
> 원본 클립: [[2026-07-14-mattpocockskills Skills for Real Engineers. Straight from my .claude directory.]]
