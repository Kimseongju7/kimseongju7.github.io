---
title: 'Headroom — LLM에 도달하기 전에 컨텍스트를 압축하는 레이어'
date: 2026-07-14 14:36:41 +0900
categories: [개발, 도구]
tags: [headroom, context-compression, llm, tokens, mcp, proxy, ai-agent, open-source]
description: '툴 출력·로그·RAG 청크·파일을 LLM에 도달하기 전에 압축하는 오픈소스 컨텍스트 압축 레이어 Headroom 정리.'
---
**Headroom**은 AI 에이전트가 읽는 모든 것 — 툴 출력, 로그, RAG 청크, 파일, 대화 히스토리 — 을 LLM에 도달하기 전에 압축하는 오픈소스(Apache 2.0) 컨텍스트 압축 레이어다. JSON 데이터는 60~95%, 코딩 에이전트는 15~20% 토큰을 줄이면서 같은 답을 얻는 것이 목표이며, 로컬 우선(local-first)이고 압축이 가역적(reversible)이다. 데모에서는 10,144 토큰이 1,260 토큰으로 줄어도 같은 FATAL 로그를 찾아냈다.

## 제공 형태

- **라이브러리** — Python/TypeScript에서 `compress(messages)`를 인라인 호출
- **프록시** — `headroom proxy --port 8787`, 코드 수정 없이 어떤 언어에서든 사용
- **에이전트 래핑** — `headroom wrap claude|codex|copilot|cursor|aider|opencode|cline|continue|goose|openhands|openclaw|vibe` 한 줄로 적용, `headroom unwrap <tool>`로 해제
- **MCP 서버** — `headroom_compress`, `headroom_retrieve`, `headroom_stats` 툴 제공
- **크로스 에이전트 메모리** — Claude·Codex·Gemini가 공유하는 저장소, 자동 중복 제거
- **`headroom learn`** — 실패한 세션을 마이닝해 교정 내용을 `CLAUDE.local.md`(기본, gitignore됨) 또는 `CLAUDE.md` / `AGENTS.md` / `GEMINI.md`에 기록
- **출력 토큰 절감** — 보내는 토큰뿐 아니라 모델이 되쓰는 토큰도 줄임(형식적 서두·코드 재출력 제거, 루틴 단계에서 딥 "thinking" 생략)
- **가역 압축(CCR)** — 원본을 로컬에 캐시해 필요 시 `headroom_retrieve`로 복원

## 동작 구조

에이전트/앱과 LLM 프로바이더(Anthropic, OpenAI, Bedrock 등) 사이에서 로컬로 동작한다: `CacheAligner → ContentRouter → CCR` 파이프라인.

- **ContentRouter** — 콘텐츠 타입을 감지해 알맞은 압축기 선택
- **SmartCrusher** — 범용 JSON 압축(딕셔너리 배열, 중첩 객체, 혼합 타입)
- **CodeCompressor** — AST 인식 코드 압축(Python, JS/TS, Go, Rust, Java, C/C++, Perl)
- **Kompress-v2-base** — 에이전트 트레이스로 학습한 자체 HuggingFace 텍스트 압축 모델
- **CacheAligner** — 프리픽스를 안정화해 프로바이더 KV 캐시가 실제로 히트하도록 함
- **라이브존(live-zone) 압축** — 새 바이트(신규 툴 출력, 최신 턴)만 압축하고 고정 프리픽스는 바이트 동일하게 유지해 캐시를 깨지 않음. 히스토리는 버리지 않는다
- 이미지 압축(학습된 ML 라우터로 40~90% 감소), SharedContext(멀티에이전트 간 압축 컨텍스트 전달)도 포함

## 시작하기 (60초)

```
# 1 — 설치
uv tool install "headroom-ai[all]"      # 전역 CLI (자체 가상환경)
pip install "headroom-ai[all]"          # Python — headroom CLI 포함
npm install headroom-ai                 # TypeScript SDK만 — CLI 없음

# 2 — 모드 선택
headroom wrap claude                    # 코딩 에이전트 래핑
headroom proxy --port 8787              # 드롭인 프록시
# 또는: from headroom import compress    # 인라인 라이브러리

# 3 — 확인
headroom doctor                         # 라우팅 동작 확인
headroom perf
headroom dashboard                      # 실시간 절감 대시보드 (프록시 실행 중이어야 함)
```

- CLI는 **PyPI 패키지에만** 포함된다. npm의 `headroom-ai`는 임포트해 쓰는 TypeScript SDK일 뿐 `headroom` 명령을 제공하지 않는다.
- Python 3.10+ 필요. 세분화된 extras: `[proxy]`, `[mcp]`, `[ml]`, `[code]`, `[memory]`, `[vector]`(HNSW 백엔드, C++ 툴체인 필요, `[all]`에 미포함), `[relevance]`, `[image]`, `[agno]`, `[langchain]`, `[evals]`, `[pytorch-mps]`(Apple GPU 오프로드) 등.
- 에이전트를 래핑하면 로컬 프록시 시작, rtk·tokensave 같은 툴을 주는 MCP 서버 설정, 프록시 경유로 구성된 에이전트 세션 실행까지 한 번에 처리된다.
- Codex 등 셸 `PATH`를 못 물려받는 MCP 클라이언트는 `uv tool install` 후 `command -v headroom`으로 얻은 절대 경로를 MCP 설정에 지정한다.

## 실측 결과

실제 에이전트 워크로드 절감:

| 워크로드 | 이전 | 이후 | 절감 |
| --- | --- | --- | --- |
| 코드 검색 (결과 100건) | 17,765 | 1,408 | **92%** |
| SRE 장애 디버깅 | 65,694 | 5,118 | **92%** |
| GitHub 이슈 트리아지 | 54,174 | 14,761 | **73%** |
| 코드베이스 탐색 | 78,502 | 41,254 | **47%** |

표준 벤치마크에서 정확도 유지:

| 벤치마크 | 분야 | N | 베이스라인 | Headroom | 차이 |
| --- | --- | --- | --- | --- | --- |
| GSM8K | 수학 | 100 | 0.870 | 0.870 | ±0.000 |
| TruthfulQA | 사실성 | 100 | 0.530 | 0.560 | +0.030 |
| SQuAD v2 | QA | 100 | — | 97% | 19% 압축 |
| BFCL | 툴 | 100 | — | 97% | 32% 압축 |

재현: `python -m headroom.evals suite --tier 1`

## 출력 토큰 절감

Opus급 모델은 출력 비용이 입력의 5배인데, 출력 상당수는 낭비다("Great, let me…" 서두, 방금 보여준 코드 재출력, 파일 읽기 같은 루틴 단계의 딥 thinking). 프록시에서 코드 변경 없이 줄일 수 있다:

- **Verbosity steering** — 시스템 프롬프트 끝에 "간결하게, 컨텍스트 재서술 금지" 노트를 덧붙임(프롬프트 캐시는 유지)
- **Effort routing** — 툴 결과(파일 읽기, 테스트 통과) 후 재개 턴은 thinking effort를 낮추고, 새 질문과 에러는 전체 effort 유지

`export HEADROOM_OUTPUT_SHAPER=1`(기본 꺼짐) 후 프록시 실행. 이 스위치는 요청마다 라이브로 읽히며, `headroom wrap`이 재사용한 프록시에는 루프백 `POST /admin/runtime-env`로 현재 설정을 핫싱크해 재시작 없이 즉시 반영된다(공유 프록시에서는 전역 적용, 마지막 설정 우선).

- `headroom learn --verbosity`(--apply로 저장) — 과거 세션에서 사용자가 실제로 보여준 간결함 선호를 학습해 자동 설정
- `headroom output-savings` — 출력 절감은 반사실적(counterfactual)이라 신뢰구간이 있는 정직한 추정치로 보고 (예: 31.7%, 95% CI 27.7~35.7%). 실측을 원하면 `export HEADROOM_OUTPUT_HOLDOUT=0.1`로 대화의 10%를 대조군으로 남길 수 있다

## 에이전트 호환성

Claude Code(`--memory`, `--code-graph`, `--1m`, `--tool-search` 지원), Codex(Claude와 메모리 공유), Aider, Copilot CLI, OpenClaw(ContextEngine 플러그인), OpenCode, Cline, Continue, Goose, OpenHands, Mistral Vibe는 `headroom wrap` 지원. Cursor는 수동 설정(프록시 시작 후 base URL 안내), Cortex Code는 라이브러리 모드만(60~65% 절감). OpenAI 호환 클라이언트는 모두 `headroom proxy`로 동작하며, MCP 네이티브는 `headroom mcp install`.

GitHub Copilot CLI 구독 트래픽도 `headroom copilot-auth login` 후 `headroom wrap copilot --subscription -- --model gpt-4o`로 로컬 프록시에 태울 수 있다(GitHub Enterprise Server는 `GITHUB_COPILOT_ENTERPRISE_DOMAIN` 설정; Docker/CI에서는 명시적 `GITHUB_COPILOT_TOKEN` 권장).

## 적합/부적합 및 통합

**적합**: AI 코딩 에이전트를 매일 쓰며 코드 변경 없이 비용을 아끼고 싶을 때, 여러 에이전트에 걸친 공유 메모리가 필요할 때, TTL 내 원본 복원이 가능한 가역 압축이 필요할 때.
**부적합**: 단일 프로바이더의 네이티브 컴팩션만으로 충분할 때, 로컬 프로세스를 못 돌리는 샌드박스 환경일 때.

통합 지점: Anthropic/OpenAI SDK(`withHeadroom(...)`), Vercel AI SDK(`headroomMiddleware()`), LiteLLM(`HeadroomCallback`), LangChain(`HeadroomChatModel`), Agno, Strands, ASGI 미들웨어, 멀티에이전트 `SharedContext` 등.

## 설치·운영 팁

- `docker pull ghcr.io/chopratejas/headroom:latest`도 가능. `[all]`은 프레임워크 어댑터(`[langchain]`, `[agno]`, `[strands]`, `[anyllm]`, `[bedrock]`)를 포함하지 않으므로 별도 설치.
- pipx 사용 시 Python 3.13 지정 권장 — 대시보드의 달러 절감 표시는 LiteLLM에 의존하는데 LiteLLM이 Python 3.14+에 설치되지 않아 3.14에서는 `$0.00`으로 표시된다.
- x86에서 ONNX 기반 기능(Magika 콘텐츠 감지, 임베딩 릴레번스)은 AVX2가 필요하며, 없으면 BM25·휴리스틱으로 자동 폴백. arm64/Apple Silicon은 해당 없음.
- `headroom update`는 설치 방식(pip/pipx/uv tool)을 감지해 알맞게 업그레이드하고, git 체크아웃·Docker·PEP 668 환경에서는 수동 절차를 안내한다.
- 기업 SSL 인스펙션 환경에서 `CERTIFICATE_VERIFY_FAILED`가 나면 Rust를 먼저 설치하거나 prebuilt wheel(`pip install --only-binary headroom-ai headroom-ai`)을 쓴다. Python 3.13의 `VERIFY_X509_STRICT`로 인한 "Basic Constraints ... not marked critical" 오류는 `HEADROOM_TLS_STRICT=0`으로 strict 플래그만 해제할 수 있다(체인 검증·서명·만료·호스트명 검사는 유지).

## 비교와 팀 도입

Headroom은 로컬 실행, 모든 콘텐츠 타입 커버, 주요 프레임워크 지원, 가역성이 강점이다. 비교 대상: RTK(CLI 출력만, 비가역 — Headroom이 번들로 포함해 하류를 추가 압축), lean-ctx(대안 컨텍스트 툴로 선택 가능), Compresr·Token Co.(호스티드 API, 비로컬·비가역), OpenAI Compaction(대화 히스토리만, 프로바이더 네이티브).

OSS는 개인 개발자용이고, 조직 단위 운영(상시 공유 배포, 중앙 설정, 조직 대시보드, SSO, 에어갭/VPC)은 hello@headroomlabs.ai로 문의하는 유료 지원·매니지드 오퍼링이 있다. 저장소 전체는 Apache 2.0으로 유지된다.

## 관련 노트

- [bytedance-deer-flow](/posts/bytedance-deer-flow/) — 요약·컴팩션으로 컨텍스트 윈도를 관리하는 슈퍼 에이전트 하네스
- [hkuds-openharness](/posts/hkuds-openharness/) — Auto-Compact로 멀티데이 세션 컨텍스트를 압축하는 하네스
- [claude-real-video](/posts/claude-real-video/) — 더 적고 의미 있는 프레임으로 컨텍스트 비용을 낮추는 영상 도구

> 원문: [headroomlabs-ai/headroom: Compress tool outputs, logs, files, and RAG chunks before they reach the LLM. 60-95% fewer tokens, same answers. Library, proxy, MCP server.](https://github.com/headroomlabs-ai/headroom)
> 원본 클립: 2026-07-14-headroomlabs-aiheadroom Compress tool outputs, logs, files, and RAG chunks before they reach the LLM. 60-95% fewer tokens, same answers. Library, proxy, MCP server.
