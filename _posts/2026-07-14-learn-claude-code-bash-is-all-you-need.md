---
title: 'shareAI-lab/learn-claude-code — Bash만 있으면 된다: 0부터 만드는 나노 Claude Code 에이전트 하니스'
date: 2026-07-14 14:03:19 +0900
categories: [학습, 아티클]
tags: [ai-agent, harness, claude-code, llm, tutorial, mcp, subagent, open-source]
description: 'Claude Code 같은 코딩 에이전트를 0부터 직접 만들어 보며 "하니스 엔지니어링"을 가르치는 20강 튜토리얼 저장소다. 핵심 주장은 에이전시는 모델 훈련에서 나오고, 우리가 만드는 것은 모델이 활동할 세계인 하니스라는 것이다.'
---
Claude Code 같은 코딩 에이전트를 0부터 직접 만들어 보며 "하니스 엔지니어링"을 가르치는 20강 튜토리얼 저장소다. 핵심 주장은 하나 — 에이전시(지각·추론·행동 능력)는 모델 훈련에서 나오고, 우리가 만드는 것은 모델이 활동할 세계인 하니스(harness)라는 것이다.

## 핵심 철학: 에이전트 = 모델 + 하니스

- **에이전시는 코드가 아니라 훈련에서 나온다.** 지각·추론·행동 능력은 수십억 번의 그래디언트 업데이트로 학습된 것이지, 바깥의 오케스트레이션 코드가 부여한 것이 아니다.
- 모델은 운전자, 하니스는 차체. 모델이 결정하고 하니스가 실행하며, 모델이 추론하고 하니스가 컨텍스트를 공급한다.
- 역사가 이를 증명한다:
  - 2013 — DeepMind DQN이 픽셀과 점수만 받아 Atari 7종을 학습, 2015년엔 49종에서 프로 테스터 수준(*Nature* 게재)
  - 2019 — OpenAI Five가 셀프플레이 4만 5천 년 분량 끝에 Dota 2 세계 챔피언 OG를 2-0으로 격파, 공개 매치 승률 99.4%(42,729판)
  - 2019 — DeepMind AlphaStar가 스타크래프트 II에서 프로를 10-1로 꺾고 유럽 서버 그랜드마스터(상위 0.15%) 달성
  - 2019 — 텐센트 쥐에위(Jueyu)가 왕자영요 5v5에서 KPL 프로 격파, 훈련 강도는 하루가 인간 440년에 해당
  - 2024–2025 — LLM 에이전트(Claude, GPT, Gemini)가 소프트웨어 엔지니어링을 재편. 구조는 이전 에이전트와 동일: 훈련된 모델 + 환경 + 지각·행동 도구

## 에이전트가 아닌 것

- 드래그앤드롭 워크플로 빌더, 노코드 "AI Agent" 플랫폼, 프롬프트 체인 오케스트레이션 라이브러리는 에이전트가 아니다.
- if-else 분기, 노드 그래프, 하드코딩된 라우팅으로 LLM API 호출을 엮는 것은 "허세 가득한 셸 스크립트"일 뿐이다.
- 절차적 로직을 아무리 쌓아도 자율적 행동은 자연 발생하지 않는다. 에이전시는 학습되는 것이지 코딩되는 것이 아니다.

## 하니스 엔지니어의 일

"에이전트를 만든다"는 말은 (1) 모델을 훈련하거나 (2) 하니스를 만들거나 둘 중 하나다. 우리 대부분의 일은 후자다.

```
Harness = Tools + Knowledge + Observation + Action Interfaces + Permissions
```

- **도구 구현** — 파일 입출력, 셸, API 호출, 브라우저 제어 등. 원자적이고 조합 가능하게, 설명은 명확하게.
- **지식 큐레이션** — 제품 문서, 아키텍처 결정 기록, 스타일 가이드. 미리 다 넣지 말고 필요할 때 로드.
- **컨텍스트 관리** — 서브에이전트 격리로 노이즈 차단, 컴팩션으로 히스토리 정리, 태스크 시스템으로 목표 지속.
- **권한 통제** — 파일 접근 샌드박스, 파괴적 작업 승인, 신뢰 경계 강제.
- **궤적 데이터 수집** — 에이전트의 모든 행동 시퀀스는 다음 세대 모델의 파인튜닝 재료다.

## 왜 Claude Code인가

Claude Code가 가장 우아한 하니스인 이유는 하지 *않는* 것에 있다 — 에이전트가 되려 하지 않고, 경직된 워크플로를 강요하지 않고, 모델의 판단을 수제 결정 트리로 대체하지 않는다. 본질만 남기면:

```
Claude Code = 하나의 에이전트 루프
            + 도구 (bash, read, write, edit, glob, grep, browser...)
            + 온디맨드 스킬 로딩 + 컨텍스트 컴팩션 + 서브에이전트 생성
            + 의존성 그래프 태스크 시스템 + 비동기 메일박스 팀 협업
            + worktree 격리 병렬 실행 + 권한 거버넌스
            + hooks 확장 + 메모리 지속 + MCP 외부 기능 라우팅
```

핵심 패턴은 단순하다. LLM을 호출하고, `stop_reason`이 `tool_use`면 도구를 실행해 결과를 messages에 붙이고 루프를 돌고, 아니면 텍스트를 반환한다. 모델이 언제 도구를 부르고 언제 멈출지 결정하고, 코드는 모델이 요청한 것을 실행할 뿐이다. 20개 레슨은 모두 이 루프 위에 하니스 메커니즘을 한 겹씩 얹는다 — 루프 자체는 절대 바뀌지 않는다.

## 20개 레슨 구성

레슨마다 메커니즘 하나와 모토 하나가 붙는다.

- **s01 Agent Loop** — "One loop & Bash is all you need": 도구 하나 + 루프 하나 = 에이전트 하나
- **s02 Tool Use** — 도구 추가는 핸들러 하나 추가일 뿐, 루프는 그대로
- **s03 Permission** — 먼저 경계를 정하고 그다음 자유를 준다
- **s04 Hooks** — 루프를 다시 쓰지 말고 루프 주변에 훅을 건다
- **s05 TodoWrite** — 계획 없는 에이전트는 표류한다; 먼저 단계를 나열하면 완수율이 배가된다
- **s06 Subagent** — 큰 일은 잘게 쪼개고, 서브태스크마다 깨끗한 컨텍스트
- **s07 Skill Loading** — 지식은 미리가 아니라 필요할 때 로드
- **s08 Context Compact** — 컨텍스트는 반드시 차오른다; 다층 컴팩션으로 무한 세션 확보
- **s09 Memory** — 선택(selection)·추출(extraction)·통합(consolidation) 3개 서브시스템
- **s10 System Prompt** — 프롬프트는 하드코딩이 아니라 런타임에 섹션 단위로 조립
- **s11 Error Recovery** — 실패하면 재시도, 공간 확보, 또는 다른 경로
- **s12 Task System** — 파일 기반 태스크 그래프(`TaskRecord` / `blockedBy` / 디스크 지속)
- **s13 Background Tasks** — 느린 작업은 백그라운드로, 완료 시 알림 주입
- **s14 Cron Scheduler** — 시간 기반 자동 트리거
- **s15 Agent Teams** — 지속적 팀원 + 비동기 메일박스(`MessageBus`)
- **s16 Team Protocols** — 고정된 요청-응답 형식으로 협업(셧다운 핸드셰이크, 계획 승인)
- **s17 Autonomous Agents** — 리더가 일일이 배정하지 않고 팀원이 보드에서 스스로 일을 집는 자기조직화
- **s18 Worktree Isolation** — 각자 자기 디렉터리에서 작업, 태스크-디렉터리를 ID로 바인딩
- **s19 MCP Plugin** — 외부 도구를 같은 도구 풀에 플러그인(멀티 트랜스포트, 채널 라우팅)
- **s20 Comprehensive** — 모든 메커니즘이 하나의 루프로 회귀

학습 경로는 행동하기 → 복잡한 작업 처리 → 기억·복구 → 장기 실행 → 다중 에이전트 협업 → 확장·조립 순이다.

## 저장소 구조와 시작 방법

- 현재 정본은 루트의 `s01_*`~`s20_*` 폴더(챕터별 README + 영어/일본어 번역 + 실행 가능한 `code.py` + SVG 다이어그램). `docs/`, `agents/`, `web/`은 구 12레슨 트랙으로 과도기용으로만 유지되며, 신구 챕터 번호가 일치하지 않으니 섞어 읽지 말 것.
- 시작: `git clone` 후 `pip install -r requirements.txt`, `.env`에 `ANTHROPIC_API_KEY` 설정, `python s01_agent_loop/code.py`부터 순서대로.
- 교육 목적상 일부 프로덕션 메커니즘(전체 훅 버스, 규칙 기반 권한, 세션 resume/fork, MCP OAuth 등)은 의도적으로 단순화·생략됐다. JSONL 메일박스 프로토콜도 교육용 구현이다.
- 라이선스: MIT

## 다음 단계와 자매 프로젝트

- **Kode Agent CLI** (`npm i -g @shareai-lab/kode`) — 오픈소스 코딩 에이전트 CLI. 스킬·LSP 지원, Windows 호환, GLM / MiniMax / DeepSeek 등 오픈 모델 지원.
- **Kode Agent SDK** — 백엔드, 브라우저 확장, 임베디드 기기 등에 에이전트 기능을 내장하는 독립 라이브러리.
- **claw0** — 자매 튜토리얼. 이 저장소가 가르치는 "쓰고 버리는" 세션형 하니스에 하트비트(30초마다 깨워 할 일 확인)와 크론(스스로 미래 작업 예약)을 더해, IM 멀티채널(WhatsApp / Telegram / Slack / Discord 등 13+), 지속 메모리, Soul 성격 시스템까지 갖춘 상시 가동형 개인 AI 비서 하니스를 처음부터 해부한다.

## 관련 노트

- [what-is-a-loop-anyway](/posts/what-is-a-loop-anyway/) — 이 튜토리얼의 "하나의 에이전트 루프"가 어떤 종류의 루프인지 지도로 정리
- [claude-code-multi-agent-sessions](/posts/claude-code-multi-agent-sessions/) — s06 서브에이전트·s15 에이전트 팀을 실제 Managed Agents API로 구현한 형태
- [hkuds-openharness](/posts/hkuds-openharness/) — 같은 "하네스 엔지니어링" 문제의식을 공유하는 오픈 하네스
- [bytedance-deer-flow](/posts/bytedance-deer-flow/) — 슈퍼에이전트 하네스의 또 다른 구현 사례
- [harness-engineering-codex](/posts/harness-engineering-codex/) — Codex 관점에서 본 하네스 엔지니어링

> 원문: [shareAI-lab/learn-claude-code: Bash is all you need - A nano claude code–like 「agent harness」, built from 0 to 1](https://github.com/shareAI-lab/learn-claude-code)
> 원본 클립: 2026-07-14-shareAI-lablearn-claude-code Bash is all you need -  A nano claude code–like 「agent harness」, built from 0 to 1
