---
title: 'bytedance/deer-flow — 리서치·코딩·창작을 수행하는 오픈소스 슈퍼 에이전트 하네스 DeerFlow 2.0'
date: 2026-07-14 14:12:42 +0900
categories: [개발, 도구]
tags: [ai-agent, harness, langgraph, langchain, sandbox, mcp, skills, deep-research, bytedance]
description: '서브에이전트·메모리·샌드박스를 오케스트레이션해 리서치·코딩·창작을 수행하는 바이트댄스의 오픈소스 슈퍼 에이전트 하네스 DeerFlow 2.0 정리.'
---
DeerFlow(**D**eep **E**xploration and **E**fficient **R**esearch **Flow**)는 **서브에이전트, 메모리, 샌드박스**를 오케스트레이션해 거의 무엇이든 해내는 오픈소스 **슈퍼 에이전트 하네스**다. 확장 가능한 스킬로 구동되며, 수분에서 수시간이 걸리는 다양한 수준의 태스크를 처리한다. 2026년 2월 28일 버전 2 출시 후 GitHub Trending 1위를 차지했다.

## Deep Research에서 슈퍼 에이전트 하네스로

DeerFlow는 Deep Research 프레임워크로 시작했지만, 커뮤니티가 데이터 파이프라인, 슬라이드 덱 생성, 대시보드, 콘텐츠 워크플로 자동화 등 예상 밖의 용도로 확장해 나갔다. 그래서 팀은 DeerFlow가 단순한 리서치 도구가 아니라 에이전트가 실제로 일을 해내게 하는 인프라, 즉 **하네스(harness)** 임을 깨닫고 처음부터 다시 만들었다.

- **DeerFlow 2.0은 완전 재작성**으로 v1과 코드를 공유하지 않는다. 기존 Deep Research 프레임워크는 [`1.x` 브랜치](https://github.com/bytedance/deer-flow/tree/main-1.x)에서 유지된다.
- LangGraph와 LangChain 위에 구축됐고, 파일시스템·메모리·스킬·샌드박스 인식 실행·서브에이전트 계획/생성까지 에이전트에 필요한 모든 것이 기본 탑재돼 있다("batteries included, fully extensible").
- 공식 웹사이트([deerflow.tech](https://deerflow.tech/))에서 실제 데모를 볼 수 있다. ByteDance Volcengine의 Coding Plan에서는 Doubao-Seed-2.0-Code, DeepSeek v3.2, Kimi 2.5 사용을 권장하며, BytePlus가 자체 개발한 검색·크롤링 툴셋 InfoQuest도 새로 통합됐다.

## 빠른 시작

- **원라인 에이전트 셋업**: Claude Code, Codex, Cursor, Windsurf 등 코딩 에이전트에 "Help me clone DeerFlow if needed, then bootstrap it for local development by following https://raw.githubusercontent.com/bytedance/deer-flow/main/Install.md" 한 문장을 건네면 된다.
- **수동 설정**: `git clone` 후 `make setup`을 실행하면 LLM 제공자, 웹 검색(선택), 샌드박스 모드·bash 접근·파일 쓰기 같은 실행/안전 설정을 안내하는 대화형 마법사가 약 2분 만에 최소 `config.yaml`과 `.env`를 생성한다. `make doctor`로 설정을 검증하고, 이슈 제보 시 `make support-bundle`로 요약/초안/증거 zip을 만들 수 있다.
- 모델 설정은 `config.yaml`에서 관리한다. OpenAI 호환 API(`langchain_openai:ChatOpenAI`), OpenRouter(`base_url` 지정), Responses API(`use_responses_api: true`), vLLM 0.19.0(`deerflow.models.vllm_provider:VllmChatModel`, Qwen류 reasoning 모델의 `enable_thinking` 토글 지원)을 지원한다.
- **CLI 기반 제공자**도 지원: Codex CLI(`~/.codex/auth.json` 사용)와 Claude Code OAuth(`CLAUDE_CODE_OAUTH_TOKEN`, `ANTHROPIC_AUTH_TOKEN`, `~/.claude/.credentials.json` 등).
- **실행**: Docker 개발은 `make docker-init` → `make docker-start`, 프로덕션은 `make up`/`make down`, 로컬 개발은 `make check` → `make install` → `make dev`. 접속은 `http://localhost:2026`.
- **배포 사이징**: 로컬 평가 4 vCPU/8GB RAM(권장 8 vCPU/16GB), Docker 개발 4 vCPU/8GB(권장 8/16), 장기 실행 서버 8 vCPU/16GB(권장 16/32). 영속 서버는 Linux + Docker 권장. 영속 배포에는 `database.backend`를 `sqlite` 또는 `postgres`로 설정한다.
- 프로덕션 Gateway는 기본 단일 워커(`GATEWAY_WORKERS=1`)이며, Redis stream bridge로 SSE 전달과 `Last-Event-ID` 재생을 워커 간 공유할 수 있다.

## 핵심 기능

### 스킬과 도구

스킬은 DeerFlow가 "거의 무엇이든" 하게 만드는 요소다. 표준 Agent Skill은 워크플로·베스트 프랙티스·참조 리소스를 정의한 마크다운 파일이며, 리서치·리포트 생성·슬라이드 제작·웹 페이지·이미지/영상 생성 등의 내장 스킬이 제공된다. 커스텀 스킬 추가·교체·조합이 가능하다.

- 스킬은 **점진적 로딩**된다 — 태스크에 필요할 때만 로드해 컨텍스트 윈도를 가볍게 유지한다.
- `/skill-name`으로 시작하는 요청(예: `/data-analysis analyze uploads/foo.csv`)으로 한 턴 동안 스킬을 명시적으로 활성화할 수 있다.
- 스킬 설치와 에이전트의 스킬 편집은 LLM 기반 스캐너 전에 결정론적 안전 스캐너 **SkillScan**을 거친다: 오프라인으로 동작하며 개인 키나 셸 실행 같은 고신뢰 `CRITICAL` 발견을 차단한다. 읽기 전용 스킬 품질 리뷰용 **skill-reviewer** 스킬도 내장돼 있다.
- 도구도 같은 철학: 웹 검색, 웹 페치, 렌더링 웹 캡처, 파일 작업, bash 실행이 코어 툴셋이고 MCP 서버·Python 함수로 커스텀 도구를 붙일 수 있다.
- 샌드박스 내 경로는 `/mnt/skills/public`(research, report-generation, slide-creation, web-page, image-generation)과 `/mnt/skills/custom`으로 구성된다.

### Claude Code 통합

`claude-to-deerflow` 스킬로 Claude Code에서 실행 중인 DeerFlow 인스턴스와 직접 상호작용할 수 있다. `npx skills add https://github.com/bytedance/deer-flow --skill claude-to-deerflow`로 설치 후 `/claude-to-deerflow` 명령을 쓴다. 메시지 전송과 스트리밍 응답, 실행 모드 선택(flash/standard/pro/ultra), 헬스 체크, 모델·스킬·에이전트 목록, 스레드 관리, 파일 업로드를 지원한다.

### 서브에이전트

리드 에이전트가 즉석에서 서브에이전트를 생성한다 — 각각 스코프된 컨텍스트·도구·종료 조건을 갖고, 가능하면 병렬 실행되며, 구조화된 결과를 보고하면 리드 에이전트가 종합한다. 리서치 태스크 하나가 열두 개의 서브에이전트로 팬아웃돼 각기 다른 각도를 탐색한 뒤 하나의 리포트·웹사이트·슬라이드 덱으로 수렴하는 식으로, 수분~수시간짜리 태스크를 처리한다. 장기 실행 서브에이전트는 오래된 이력을 컴팩션해 요약을 재주입하고, 모델 요청 실패는 실패한 태스크로 정확히 보고된다.

### 샌드박스와 파일시스템

DeerFlow는 자기만의 컴퓨터를 갖는다. 태스크마다 스킬·워크스페이스·업로드·아웃풋의 전체 파일시스템 뷰를 가진 실행 환경이 주어지고, 에이전트는 파일을 읽고 쓰고 편집하며 이미지를 보고 (안전하게 설정된 경우) 셸 명령을 실행한다.

- 샌드박스 모드: **Local**(호스트 직접 실행), **Docker**(격리 컨테이너), **Kubernetes**(provisioner 서비스 경유 파드).
- `AioSandboxProvider`는 격리 컨테이너에서 셸을 실행하고, `LocalSandboxProvider`는 보안 격리 경계가 아니므로 호스트 bash가 기본 비활성이다.
- 실행 후 `workspace`/`outputs` 디렉터리의 변경 요약이 기록되고 Web UI에 "files changed" 배지로 표시된다. 컨테이너 내 경로는 `/mnt/user-data/uploads·workspace·outputs`.

### 컨텍스트 엔지니어링

- **격리된 서브에이전트 컨텍스트**: 서브에이전트는 메인 에이전트나 다른 서브에이전트의 컨텍스트를 볼 수 없어 자기 태스크에 집중한다.
- **요약(Summarization)**: 완료된 서브태스크 요약, 중간 결과의 파일시스템 오프로딩, 관련성 낮아진 내용 압축으로 긴 멀티스텝 태스크에서도 컨텍스트 윈도를 지킨다.
- **엄격한 툴콜 복구**: 공급자/미들웨어가 툴콜 루프를 중단시키면 강제 중단 메시지의 raw 툴콜 메타데이터를 제거하고 끊긴 호출에 플레이스홀더 결과를 주입해, `tool_call_id` 시퀀스를 엄격히 검증하는 OpenAI 호환 reasoning 모델의 이력 오류를 막는다.

### 장기 기억

대부분의 에이전트는 대화가 끝나면 모두 잊지만 DeerFlow는 기억한다. 세션을 넘어 프로필·선호·축적된 지식(글쓰기 스타일, 기술 스택, 반복 워크플로)의 영속 메모리를 구축한다. 메모리는 로컬에 저장돼 사용자 통제하에 있고, 중복 사실 항목은 적용 시점에 걸러진다.

## 세션 목표, 컴팩션, 스케줄 태스크

- **`/goal <완료 조건>`**: 현재 스레드에 하나의 활성 완료 조건을 부착한다. 스킬 활성화가 아닌 스레드 스코프 상태로, 충족되거나 지울 때까지 턴을 넘어 유지된다. 매 실행 후 비-thinking 평가자 모델이 대화를 목표와 대조해 타입 있는 blocker(`missing_evidence`, `needs_user_input`, `run_failed`, `external_wait`, `goal_not_met_yet`)를 반환하고, 조건을 만족하면 숨은 continuation을 주입한다(기본 상한 8회, 동일 무진전 평가 2회면 중단). `/goal`(현재 목표 표시), `/goal clear` 지원.
- **`/compact`**: 오래된 컨텍스트를 요약해, 전체 대화는 보이되 이후 모델 호출에는 요약 + 최근 메시지를 쓴다.
- **스케줄 태스크 MVP**: `/workspace/scheduled-tasks`에서 관리. `once`/`cron` 스케줄, 스레드 재사용 여부 선택, 겹칠 때 `skip` 동작, 일시정지/재개/수동 트리거/이력/삭제 지원. `scheduler.enabled`로 백그라운드 폴링을 켠다. 아직 대화 중 생성 도구·알림 전용 잡·채널/GitHub 디스패치·`interval` 타입은 없다.

## IM 채널

메시징 앱에서 태스크를 받을 수 있고, 어느 채널도 공인 IP가 필요 없다. Telegram(Bot API 롱폴링, 쉬움), Slack(Socket Mode), Feishu/Lark(WebSocket), WeChat(Tencent iLink), WeCom(WebSocket), DingTalk(Stream Push)을 지원하며 채널별 설정 난이도는 Moderate 수준. `channel_connections`를 켜면 로그인 사용자가 워크스페이스 UI 사이드바에서 직접 채널을 바인딩할 수 있다. 채널 연결 후에는 채팅에서 `/new`, `/status`, `/models`, `/memory`, `/help` 명령을 쓸 수 있고, 명령 없는 메시지는 일반 대화로 처리된다.

## 관측(Observability)

- **LangSmith**: `.env`에 `LANGSMITH_TRACING=true` 등 설정 시 모든 LLM 호출·에이전트 실행·도구 실행이 트레이싱된다.
- **Langfuse**: LangChain 호환 실행 지원. `session_id`(=thread_id), `user_id`, `trace_name`, `tags`, `metadata.deerflow_trace_id` 등 예약 트레이스 속성이 자동 주입된다.
- **Monocle**: OpenTelemetry 기반 에이전트 트레이서. 실행별 트레이스 파일을 `.monocle/`에 쓰고 VS Code 확장에서 열람하며, Okahu 등 원격 익스포터도 지원한다.
- 셋 다 동시에 켤 수 있다. Gateway 요청 트레이스 상관(`X-Trace-Id`)은 기본 비활성이며 `logging.enhance.enabled: true`로 켠다.

## 기타 사용 방식

- **임베디드 Python 클라이언트**: HTTP 서비스 없이 `DeerFlowClient`로 인프로세스에서 chat/stream/모델·스킬 관리/파일 업로드/goal 관리 등 Gateway와 같은 스키마의 API를 쓸 수 있다. CI(`TestGatewayConformance`)로 HTTP API 스키마와의 동기화가 검증된다.
- **터미널 워크벤치(TUI)**: `deerflow` 명령은 Gateway·프론트엔드·nginx·Docker 없이 `DeerFlowClient` 위에서 임베디드로 돌아가는 터미널 UI다. 같은 `config.yaml`·체크포인터·스킬·메모리·MCP·샌드박스 설정을 그대로 쓰고, `--continue`/`--resume`/`--print`/`--json` 옵션과 슬래시 커맨드 팔레트, `/goal`·`/model`·`/threads`를 지원한다. TUI 세션은 Web UI 사이드바에도 나타난다.

## 권장 모델

모델 불가지론적(model-agnostic)으로 OpenAI 호환 API를 구현한 어떤 LLM과도 동작하지만, **긴 컨텍스트(100k+ 토큰)**, **추론(reasoning) 능력**, **멀티모달 입력**, **강한 도구 사용** 능력을 갖춘 모델에서 가장 좋은 성능을 낸다.

## 보안 주의사항

DeerFlow는 시스템 명령 실행·리소스 조작·비즈니스 로직 호출 등 고권한 능력을 갖고 있어, 기본적으로 **로컬 신뢰 환경(127.0.0.1 루프백만 접근 가능)** 배포를 전제로 설계됐다. LAN·퍼블릭 클라우드 등 신뢰할 수 없는 환경에 보안 조치 없이 배포하면 무단 호출과 법적·컴플라이언스 위험이 생길 수 있다. 크로스 네트워크 배포가 필요하면 IP 허용목록(iptables/ACL), 인증 게이트웨이(nginx 리버스 프록시 + 강한 사전 인증), 네트워크 격리(전용 VLAN), 보안 업데이트 추적 같은 조치를 반드시 적용해야 한다.

## 라이선스와 감사

MIT 라이선스. LangChain과 LangGraph 위에 구축됐으며, 핵심 저자는 [Daniel Walnut](https://github.com/hetaoBackend/)과 [Henry Li](https://github.com/magiccube/)다.

## 관련 노트

- [hkuds-openharness](/posts/hkuds-openharness/) — 또 다른 오픈 에이전트 하네스 인프라(+개인 에이전트 ohmo)
- [ouroboros-agent-os](/posts/ouroboros-agent-os/) — 명세 우선 워크플로를 얹은 Agent OS
- [headroom-context-compression](/posts/headroom-context-compression/) — 요약·컴팩션과 맞닿은 컨텍스트 압축 레이어
- [openobserve](/posts/openobserve/) — OpenTelemetry 기반 관측성 플랫폼(트레이싱 대상)

> 원문: [bytedance/deer-flow: An open-source long-horizon SuperAgent harness that researches, codes, and creates.](https://github.com/bytedance/deer-flow)
> 원본 클립: 2026-07-14-bytedancedeer-flow An open-source long-horizon SuperAgent harness that researches, codes, and creates. With the help of sandboxes, memories, tools, skill, subagents and message gateway, it handles different levels of tasks that could
