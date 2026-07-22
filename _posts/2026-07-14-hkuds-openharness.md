---
title: 'HKUDS/OpenHarness — 오픈 에이전트 하네스와 내장 개인 에이전트 ohmo'
date: 2026-07-14 14:22:11 +0900
categories: [개발, 도구]
tags: [openharness, ohmo, agent-harness, mcp, claude-code, codex, python, multi-agent]
description: '도구 사용·스킬·메모리·멀티 에이전트 조율을 제공하는 오픈소스 에이전트 하네스 OpenHarness와 개인 에이전트 ohmo 정리.'
---
**OpenHarness**는 도구 사용(tool-use), 스킬, 메모리, 멀티 에이전트 조율이라는 경량 에이전트 코어 인프라를 제공하는 오픈소스 Python 프로젝트다. 그 위에 지어진 **ohmo**는 또 하나의 챗봇이 아니라 긴 세션에 걸쳐 실제로 일해주는 개인 AI 에이전트로, Feishu/Slack/Telegram/Discord에서 대화하면 스스로 브랜치를 포크하고 코드를 쓰고 테스트를 돌리고 PR을 연다. 기존 Claude Code 또는 Codex 구독으로 동작해 별도 API 키가 필요 없다.

## Agent Harness란 무엇인가

**Agent Harness**는 LLM을 감싸 실제 동작하는 에이전트로 만드는 완전한 인프라다. 모델이 지능을 제공하면, 하네스는 **손, 눈, 기억, 안전 경계**를 제공한다. OpenHarness는 연구자·빌더·커뮤니티를 위해 다음을 목표로 한다: 프로덕션 AI 에이전트의 내부 동작 **이해**, 최신 도구·스킬·조율 패턴 **실험**, 커스텀 플러그인·프로바이더·도메인 지식으로 **확장**, 검증된 아키텍처 위에 특화 에이전트 **구축**.

핵심 하네스 기능 다섯 축:

- **Agent Loop** — 스트리밍 툴콜 사이클, 지수 백오프 API 재시도, 병렬 도구 실행, 토큰 카운팅·비용 추적
- **Harness Toolkit** — 43개 도구(파일, 셸, 검색, 웹, MCP), 온디맨드 스킬 로딩(.md), 플러그인 생태계, anthropics/skills 및 플러그인 호환
- **Context & Memory** — CLAUDE.md 발견·주입, 컨텍스트 압축(Auto-Compact), MEMORY.md 영속 메모리, 세션 재개·히스토리
- **Governance** — 다단계 권한 모드, 경로/명령 수준 규칙, PreToolUse/PostToolUse 훅, 대화형 승인 다이얼로그
- **Swarm Coordination** — 서브에이전트 스폰·위임, 팀 레지스트리·태스크 관리, 백그라운드 태스크 라이프사이클, ClawTeam 연동(로드맵)

## 빠른 시작

```
# Linux / macOS / WSL 원클릭 설치
curl -fsSL https://raw.githubusercontent.com/HKUDS/OpenHarness/main/scripts/install.sh | bash
# 또는
pip install openharness-ai
```

Windows도 네이티브 지원(PowerShell 설치 스크립트). 단 PowerShell에서는 `oh`가 내장 `Out-Host` alias와 겹칠 수 있어 `openh`를 쓴다. 이후 `oh setup`(대화형 마법사)으로 프로바이더 선택·인증을 마치고 `oh`로 실행한다. **Claude / OpenAI / Copilot / Codex / Moonshot(Kimi) / GLM / MiniMax / NVIDIA NIM** 및 호환 엔드포인트를 지원한다.

비대화형 모드도 있다: `oh -p "Explain this codebase"`(stdout), `--output-format json`, `--output-format stream-json`(실시간 스트림).

### Dry Run (안전 미리보기)

`oh --dry-run`은 모델·도구·서브에이전트를 실행하지 않고 해석된 런타임 설정, 인증 상태, 스킬, 명령, 도구, MCP 서버 구성을 미리 보여준다. `ready` / `warning` / `blocked` 준비도 판정과 함께 `oh auth login` 실행, 깨진 MCP 설정 수정 같은 구체적 다음 단계를 제안한다. 의도적으로 정적이다: 모델 호출 없음, 도구 실행·서브에이전트 스폰 없음, MCP 서버 연결 없음.

## 프로바이더 호환성

프로바이더를 이름 있는 프로필로 뒷받침되는 **워크플로**로 다룬다(`oh provider list` / `oh provider use <profile>`). 내장 워크플로: Anthropic-Compatible API(Claude 공식, Kimi, GLM, MiniMax), **Claude Subscription**(로컬 `~/.claude/.credentials.json` 브리지), OpenAI-Compatible API(OpenAI, OpenRouter, DashScope, DeepSeek, SiliconFlow, Groq, Ollama, GitHub Models 등), **Codex Subscription**(`~/.codex/auth.json` 브리지), GitHub Copilot(OAuth 디바이스 플로, API 키 불요, 단기 세션 토큰 자동 갱신, `--github-domain`으로 Enterprise 지원).

`oh provider add`로 커스텀 호환 엔드포인트를 추가할 수 있고, 프로필별로 자격 증명을 바인딩해 같은 형식의 백엔드끼리 API 키를 공유하도록 강제하지 않는다. Ollama의 OpenAI 호환 엔드포인트(`http://localhost:11434/v1`)로 로컬 모델도 돌릴 수 있다.

## 하네스 아키텍처

10개 서브시스템으로 Agent Harness 패턴을 구현한다: engine(에이전트 루프), tools(43개 도구), skills(온디맨드 지식), plugins, permissions, hooks, commands(54개 슬래시 명령), mcp, memory, tasks, coordinator(멀티 에이전트), prompts, config, ui(React TUI).

하네스의 심장인 에이전트 루프는 단순하다:

```
while True:
    response = await api.stream(messages, tools)
    if response.stop_reason != "tool_use":
        break
    for tool_call in response.tool_uses:
        # 권한 검사 → 훅 → 실행 → 훅 → 결과
        result = await harness.execute_tool(tool_call)
    messages.append(tool_results)
```

모델이 **무엇을** 할지 결정하고, 하네스가 **어떻게**를 안전하고 효율적으로, 완전한 관찰 가능성과 함께 처리한다. 모든 도구는 Pydantic 입력 검증, 자기 기술(self-describing) JSON Schema, 권한 통합, 훅 지원을 갖춘다.

## 스킬·플러그인 시스템

스킬은 모델이 필요할 때만 로드되는 **온디맨드 지식**이다(commit, review, debug, plan, test, simplify, pdf, xlsx 등 40+). 사용자 레벨은 `~/.openharness/skills/`, `~/.claude/skills/`, `~/.agents/skills/`에서, 프로젝트 레벨은 현재 디렉터리에서 git 루트까지 올라가며 발견된다. 신뢰하지 않는 저장소에서는 `oh config set allow_project_skills false`로 프로젝트 스킬을 끌 수 있다. **anthropics/skills의 SKILL.md 디렉터리 레이아웃과 호환**되고, 플러그인 시스템도 **claude-code 공식 플러그인과 호환**되어 commit-commands, security-guidance, hookify, feature-dev, code-review, pr-review-toolkit 등 공식 플러그인 12개로 테스트됐다.

웹 검색은 기본 DuckDuckGo HTML 검색이며 `OPENHARNESS_WEB_SEARCH_URL`로 SearXNG 등으로 교체 가능하다. `web_search`/`web_fetch`는 SSRF 안전을 위해 `trust_env=False`를 유지해 `HTTP_PROXY`를 자동 상속하지 않고, 필요하면 `OPENHARNESS_WEB_PROXY`로 명시적으로 옵트인한다(HTTP/HTTPS만, 내장 자격 증명 불가).

## 권한과 UI

| 모드 | 동작 | 용도 |
| --- | --- | --- |
| **Default** | 쓰기/실행 전 질문 | 일상 개발 |
| **Auto** | 전부 허용 | 샌드박스 환경 |
| **Plan Mode** | 모든 쓰기 차단 | 대규모 리팩터링, 선검토 |

`settings.json`의 경로 수준 규칙(`path_rules`)과 금지 명령(`denied_commands`)으로 세밀하게 제어한다. React/Ink 기반 TUI는 명령 피커, 권한 다이얼로그, 모드 전환, 세션 재개(`/resume`), 실시간 스피너, 문맥 인식 단축키를 제공한다.

## ohmo 개인 에이전트

```
ohmo init             # ~/.ohmo 워크스페이스 초기화
ohmo config           # 채널·프로바이더 설정
ohmo gateway start    # 게이트웨이 시작 — 채팅 앱에서 ohmo 가동
```

`~/.ohmo/` 워크스페이스에는 `soul.md`(장기 성격·행동), `identity.md`(정체성), `user.md`(사용자 프로필·선호), `BOOTSTRAP.md`(첫 실행 의식), `memory/`(개인 메모리), `gateway.json`(프로바이더·채널 설정)이 있다. 채널은 현재 Telegram, Slack, Discord, Feishu 설정을 안내한다. `ohmo config`는 `oh setup`과 같은 워크플로 언어를 써서 Anthropic 호환 API, Claude 구독, OpenAI 호환 API, Codex 구독, GitHub Copilot 중 어디로든 게이트웨이를 연결할 수 있다.

## 릴리스 흐름 (What's New)

- **v0.1.0** (2026-04-01) 최초 오픈소스 릴리스 → **v0.1.2** `oh setup` 워크플로화 + ohmo 앱 패키징 → **v0.1.4** 멀티 프로바이더 인증·Moonshot/Kimi 네이티브(`reasoning_content` 지원), PermissionChecker 민감 경로 보호 → **v0.1.5** MCP HTTP 트랜스포트·자동 재연결, JSON Schema 타입 자동 추론 → **v0.1.6** Auto-Compaction으로 컨텍스트 압축을 넘는 멀티데이 세션, React TUI 마크다운 렌더링 → **v0.1.7** (2026-04-18) 설치 스크립트가 PATH 대신 `~/.local/bin` 링크 사용(Conda 셸 충돌 방지), Shift+Enter 줄바꿈.
- 미공개: `--dry-run` 안전 미리보기와 준비도 판정.

테스트는 유닛+통합 114개, CLI 플래그 E2E, 하네스 기능 E2E, React TUI E2E, 실제 anthropics/skills·claude-code 플러그인 E2E까지 전부 통과 상태로 표기돼 있다. 커스텀 도구(BaseTool 상속 + Pydantic 입력 모델), 커스텀 스킬(SKILL.md), 커스텀 플러그인(plugin.json)으로 확장할 수 있다. MIT 라이선스.

슬로건: *"The model is the agent. The code is the harness."*

## 관련 노트

- [bytedance-deer-flow](/posts/bytedance-deer-flow/) — 서브에이전트·샌드박스를 조율하는 슈퍼 에이전트 하네스
- [ouroboros-agent-os](/posts/ouroboros-agent-os/) — 명세 우선 워크플로를 얹은 Agent OS
- [learn-claude-code-bash-is-all-you-need](/posts/learn-claude-code-bash-is-all-you-need/) — 최소 나노 에이전트 하네스 관점
- [headroom-context-compression](/posts/headroom-context-compression/) — Auto-Compact와 맞닿은 컨텍스트 압축 레이어
- [harness-engineering-codex](/posts/harness-engineering-codex/) — 하네스가 에이전트의 환경·피드백 루프를 만든다는 하네스 엔지니어링 관점의 원전 격 글

> 원문: [HKUDS/OpenHarness: "OpenHarness: Open Agent Harness with a Built-in Personal Agent--Ohmo!"](https://github.com/HKUDS/OpenHarness)
> 원본 클립: 2026-07-14-HKUDSOpenHarness "OpenHarness Open Agent Harness with a Built-in Personal Agent--Ohmo!"
