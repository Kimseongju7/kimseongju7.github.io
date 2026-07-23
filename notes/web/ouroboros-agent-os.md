---
title: "Q00/ouroboros — 프롬프트를 멈추고 명세를 시작하는 Agent OS"
date: 2026-07-14 14:21:14 +0900
categories: [개발, 도구]
tags: [ouroboros, agent-os, spec-first, claude-code, codex, mcp, workflow]
slug: ouroboros-agent-os
publish: true
---
**Ouroboros**는 "Stop prompting. Start specifying."을 내건 AI 코딩용 **Agent OS**다. 비결정적인 에이전트 작업을 재생 가능(replayable)하고 관찰 가능하며 정책에 묶인 실행 계약으로 바꾸는 로컬 우선 런타임 계층으로, 임기응변식 프롬프팅을 인터뷰 → 결정화 → 실행 → 평가 → 진화의 명세 우선 워크플로로 대체한다. Claude Code, Codex CLI, OpenCode, Hermes, Gemini, Kiro, Copilot, Pi에서 동작한다.

## Agent OS 스택: 3개 저장소, 하나의 스택

여느 OS처럼 안정적인 **OS 계층**(프리미티브), **애플리케이션 계층**(도메인 워크플로), 사람이 실제로 앉는 **셸**로 나뉜다.

| 계층 | 저장소 | 역할 |
| --- | --- | --- |
| **Shell** (터미널 클라이언트) | `Q00/ourocode` | Claude/Codex/Gemini CLI를 한 세션에서 돌리는 네이티브 터미널 UI — TUI, wonderTool 결정 피커, MCP 패널 상태 |
| **Apps** (도메인 워크플로) | `Q00/ouroboros-plugins` | UserLevel 플러그인 계약 — 코어 프리미티브를 설치 가능한 도메인 프로그램(PR 운영, Jira 동기화, 인시던트, 릴리스)으로 조합 |
| **OS** (본 저장소) | `Q00/ouroboros` | Agent OS 코어 — Seed, Ledger, Runtime, MCP, 안전 경계. `ooo` 명령과 spec-first 워크플로 엔진 |

커널(`ouroboros`)이 계약을 소유한다: 어느 LLM이 실행하든 모든 행동은 Seed에 묶이고 원장(ledger)에 기록되는 재생 가능한 이벤트가 된다. 플러그인은 그 계약에 대해 범위가 지정된 능력을 선언해 도메인 워크플로가 일회성 프롬프트가 아니라 감사 가능하고 정책에 묶인 상태를 유지한다. 참고로 이 프로젝트는 **어떤 암호화폐·토큰·밈코인과도 무관**하다(pump.fun 등의 "ouroboros" 티커 포함).

## 왜 Ouroboros인가

대부분의 AI 코딩은 출력이 아니라 **입력**에서 실패한다. 병목은 AI 능력이 아니라 인간의 명확성이다.

| 문제 | 벌어지는 일 | Ouroboros의 해법 |
| --- | --- | --- |
| 모호한 프롬프트 | AI가 추측하고, 당신이 재작업 | 소크라테스식 인터뷰가 숨은 가정을 드러냄 |
| 명세 없음 | 빌드 중 아키텍처 표류 | 불변(immutable) Seed 스펙이 코드 전에 의도를 잠금 |
| 수동 QA | "괜찮아 보임"은 검증이 아님 | 3단계 자동 평가 게이트 |

## 빠른 시작

```
curl -fsSL https://raw.githubusercontent.com/Q00/ouroboros/main/scripts/install.sh | bash
```

설치 후 AI 코딩 에이전트를 열고 `ooo interview "I want to build a task management CLI"`처럼 시작한다. 설치기는 Claude Code, Codex CLI, Hermes CLI를 자동 감지해 MCP 서버를 등록하고, OpenCode/Kiro/Copilot/Gemini/Pi는 `ouroboros setup --runtime <이름>`으로 설정한다. Copilot CLI 런타임은 GitHub Copilot 모델 API로 모델 카탈로그를 실시간 발견해 기본값을 고르게 해준다.

다른 설치 경로:

- **Claude Code 플러그인만**: `claude plugin marketplace add Q00/ouroboros && claude plugin install ouroboros@ouroboros` 후 세션 안에서 `ooo setup`
- **pip/uv/pipx**: `pip install ouroboros-ai`에 `[claude]`, `[litellm]`, `[mcp]`, `[tui]`, `[all]` extras. `[dashboard]`는 호환용 no-op alias.
- **Python 3.12 이상 필요**. 제거는 `ouroboros uninstall`로 설정·MCP 등록·데이터를 전부 지운다.

한 루프를 돌고 나면: Interview가 숨은 가정 12개를 드러내고(모호성 0.19로 채점), Seed가 수용 기준·온톨로지·제약을 가진 불변 명세를 만들고, Run이 Double Diamond 분해로 실행하고, Evaluate가 Mechanical → Semantic → Multi-Model Consensus 3단계로 검증한다.

## 순환 구조 (The Loop)

자기 꼬리를 삼키는 뱀은 장식이 아니라 아키텍처 그 자체다.

```
Interview -> Seed -> Execute -> Evaluate
    ^                           |
    +---- Evolutionary Loop ----+
```

| 단계 | 내용 |
| --- | --- |
| **Interview** | 소크라테스식 질문으로 숨은 가정 노출 |
| **Seed** | 답변을 불변 명세로 결정화 |
| **Execute** | Double Diamond: Discover → Define → Design → Deliver |
| **Evaluate** | 3단계 게이트: Mechanical($0) → Semantic → Multi-Model Consensus |
| **Evolve** | Wonder("아직 모르는 게 뭐지?") → Reflect → 다음 세대 |

각 순환은 반복이 아니라 **진화**다. 평가의 출력이 다음 세대 Seed 명세의 입력이 되고, 온톨로지 유사도가 0.95 이상이면 수렴한다. `ooo ralph`는 수렴까지 세션 경계를 넘어 루프를 지속한다 — 각 단계는 무상태이고 EventStore가 전체 계보를 재구성하므로 머신이 재시작돼도 이어진다.

## 두 개의 수학적 게이트

**모호성 점수(Ambiguity Score)** — Interview는 느낌이 아니라 수학이 준비됐다고 말할 때 끝난다. `Ambiguity = 1 - Σ(clarity_i × weight_i)`로, 각 차원을 LLM이 0.0~1.0으로 채점(재현성 위해 temperature 0.1)하고 가중치를 곱한다: 목표 명확도(Greenfield 40%/Brownfield 35%), 제약 명확도(30%/25%), 성공 기준(30%/25%), 컨텍스트 명확도(Brownfield만 15%). **임계값 Ambiguity ≤ 0.2**를 통과해야 Seed를 만들 수 있다. 왜 0.2인가 — 가중 명확도 80%면 남은 미지가 코드 수준 결정으로 풀 만큼 작기 때문이고, 그 위에서는 아직 아키텍처를 추측하는 단계라서다.

**온톨로지 수렴** — 연속 세대가 온톨로지적으로 같은 스키마를 내면 루프가 멈춘다. `Similarity = 0.5 × name_overlap + 0.3 × type_match + 0.2 × exact_match`, **임계값 Similarity ≥ 0.95**. 유사도 외에 병리적 패턴도 감지한다: 정체(3세대 연속 ≥ 0.95), 진동(Gen N ≈ Gen N-2), 반복 피드백(3세대에 걸쳐 질문 중복 ≥ 70%), 하드 캡(30세대).

## 명령어

에이전트 세션 안에서는 `ooo <cmd>` 스킬을, 터미널에서는 `ouroboros` CLI를 쓴다.

| 스킬 | CLI 대응 | 기능 |
| --- | --- | --- |
| `ooo setup` | `ouroboros setup` | 런타임 등록·프로젝트 설정(1회) |
| `ooo interview` | `ouroboros init start` | 소크라테스식 질문 |
| `ooo auto` | `ouroboros auto` | 목표 → A-grade Seed → 제한된 루프의 실행 핸드오프 |
| `ooo seed` | (인터뷰가 생성) | 불변 스펙으로 결정화 |
| `ooo run` | `ouroboros run seed.yaml` | Double Diamond 분해로 실행 |
| `ooo evaluate` / `ooo evolve` / `ooo unstuck` / `ooo ralph` | (MCP 경유) | 3단계 검증 / 진화 루프 / 5개 수평 사고 페르소나 / 지속 루프 |
| `ooo status` / `ooo resume-session` / `ooo cancel` | `ouroboros status ...` / `ouroboros resume` / `ouroboros cancel ...` | 세션 추적·재연결·취소 |
| `ooo pm` / `ooo qa` / `ooo brownfield` / `ooo publish` | (MCP/skill) | PM 인터뷰+PRD / 범용 QA 판정 / 기존 저장소 스캔 / Seed를 GitHub Epic/Task 이슈로 발행 |
| `ooo tutorial` / `ooo help` / `ooo update` | — | 대화형 실습 / 참조 / 업데이트 |

`/resume`은 Claude Code 내장 세션 피커에 예약돼 있으므로 Ouroboros 세션에는 `ooo resume-session`을 쓴다.

## 아홉 개의 마음 (The Nine Minds)

각기 다른 사고 방식의 에이전트 9개가 필요할 때만 로드된다: Socratic Interviewer("무엇을 가정하고 있나?"), Ontologist("이게 정말 무엇인가?"), Seed Architect("완전하고 모호하지 않은가?"), Evaluator("맞는 걸 만들었나?"), Contrarian("반대가 사실이라면?"), Hacker("어떤 제약이 진짜인가?"), Simplifier("동작하는 가장 단순한 것은?"), Researcher("실제 증거가 있나?"), Architect("처음부터 다시 지으면 이렇게 지을까?").

## 내부 구조

Python 3.12+ 기준 `src/ouroboros/`는 bigbang(인터뷰·모호성 채점·brownfield 탐색), routing(PAL Router), execution(Double Diamond·계층적 AC 분해), evaluation, evolution, resilience, observability, persistence(SQLAlchemy + aiosqlite 이벤트 소싱·체크포인트), orchestrator(런타임 추상화), core, providers(LiteLLM 어댑터, 100+ 모델), mcp, plugin, tui, cli로 구성된다.

핵심 내부:

- **PAL Router** — Frugal(1x) → Standard(10x) → Frontier(30x) 3단계 비용 최적화, 실패 시 자동 상향·성공 시 자동 하향
- **Drift** — Goal 50% + Constraint 30% + Ontology 20% 가중 측정, 임계값 ≤ 0.3
- **Brownfield** — 여러 언어 생태계의 설정 파일 자동 감지
- **Evolution** — 최대 30세대, 온톨로지 유사도 ≥ 0.95에서 수렴
- **Stagnation** — 스핀, 진동, 무드리프트, 수확 체감 패턴 감지
- **런타임 백엔드** — 플러그블 추상화(`orchestrator.runtime_backend`)로 Claude Code, Codex CLI, OpenCode, Hermes, Gemini, Goose, Kiro, Copilot, Pi를 1급 지원; 같은 워크플로 스펙, 다른 실행 엔진

## Wonder에서 온톨로지로

철학적 엔진: 모든 좋은 질문은 더 깊은 질문으로 이어지고, 그 깊은 질문은 언제나 **온톨로지적**이다 — "어떻게 하지?"가 아니라 "이게 정말 무엇이지?". "Task CLI를 만들자"는 "Task란 무엇인가? Priority란 무엇인가?"가 된다. "Task가 무엇인가"(삭제 가능한가 보관 가능한가? 개인용인가 팀용인가?)에 답하면 재작업의 한 부류 전체가 사라진다 — **온톨로지 질문이 가장 실용적인 질문이다.** 이를 Double Diamond로 구현한다: 첫 다이아몬드는 소크라테스적(질문으로 발산, 온톨로지적 명확성으로 수렴), 두 번째는 실용적(설계 옵션으로 발산, 검증된 전달로 수렴). 이해하지 못한 것은 설계할 수 없다.

두 게이트, 하나의 철학: **명확해질 때까지 만들지 말고(Ambiguity ≤ 0.2), 안정될 때까지 진화를 멈추지 마라(Similarity ≥ 0.95).** "뱀은 반복하지 않는다 — 진화한다." MIT 라이선스.

같은 프로젝트의 한국어 README 정리: [[ouroboros-readme-ko]]

## 관련 노트

- [[hkuds-openharness]] — 또 다른 오픈 에이전트 하네스(Agent Harness) 인프라
- [[bytedance-deer-flow]] — 서브에이전트·샌드박스를 조율하는 슈퍼 에이전트 하네스
- [[mattpocock-skills]] — 그릴링/스펙 우선(to-spec)으로 요구를 결정화하는 스킬 접근
- [[what-is-a-loop-anyway]] — `ooo ralph`가 구현하는 태스크 루프(Ralph Loop) 개념을 지도로 정리한 글

> 원문: [Q00/ouroboros: Agent OS: Stop prompting. Start specifying.](https://github.com/Q00/ouroboros)
> 원본 클립: [[2026-07-14-Q00ouroboros Agent OS Stop prompting. Start specifying.]]
