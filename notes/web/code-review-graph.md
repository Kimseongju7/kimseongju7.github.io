---
title: "code-review-graph — AI 코딩 도구가 필요한 파일만 읽게 하는 로컬 코드 지능 그래프"
date: 2026-07-14 14:07:53 +0900
categories: [개발, 도구]
tags: [mcp, code-review, tree-sitter, tokens, ai, cli, developer-tools, github-action]
slug: code-review-graph
publish: true
---
`tirth8205/code-review-graph`는 Tree-sitter로 코드베이스의 구조적 지도를 만들어 두고, MCP를 통해 AI 코딩 도구에게 정확한 컨텍스트만 넘겨주는 로컬 우선(local-first) 코드 지능 그래프다. 리뷰 작업에서 코드베이스 전체를 다시 읽는 토큰 낭비를 줄여, 6개 실제 저장소 벤치마크에서 질문당 중앙값 약 82배(최대 528배)의 토큰 절감을 기록했다.

## 문제의식과 동작 방식

- AI 코딩 도구는 리뷰 작업에서 코드베이스의 큰 부분을 반복해서 다시 읽게 되곤 한다. 이 도구는 [Tree-sitter](https://tree-sitter.github.io/tree-sitter/)로 코드의 구조적 지도를 만들고, 변경을 증분(incremental)으로 추적하며, [MCP](https://modelcontextprotocol.io/)로 AI 어시스턴트에 정밀한 컨텍스트를 제공한다.
- 저장소를 AST로 파싱해 노드(함수·클래스·임포트)와 엣지(호출·상속·테스트 커버리지)의 그래프로 SQLite에 저장하고, 리뷰 시점에 AI가 읽어야 할 **최소 파일 집합**을 계산한다.
- **블라스트 레이디어스(blast radius) 분석**: 파일이 바뀌면 영향받을 수 있는 모든 호출자·의존 코드·테스트를 그래프로 추적한다. AI는 전체 프로젝트 대신 이 파일들만 읽는다.
- **증분 업데이트**: 훅/워치 모드가 켜져 있으면 파일 저장 시 변경 파일만 diff하고 SHA-256 해시로 의존 파일을 찾아 재파싱한다. 2,900파일 프로젝트가 2초 안에 재인덱싱된다.
- **모노레포**: 토큰 낭비가 가장 심한 대형 모노레포에서 27,700개 이상 파일을 리뷰 컨텍스트에서 제외하고 실제로는 ~15개 파일만 읽게 한다.

## 빠른 시작

```
pip install code-review-graph      # 또는 pipx install code-review-graph
code-review-graph install          # 지원 플랫폼 자동 감지·설정
code-review-graph build            # 코드베이스 파싱
```

- `install` 한 방으로 설치된 AI 코딩 도구를 감지해 MCP 설정, 플랫폼 네이티브 훅/스킬 설치, 그래프 인식 지침 주입까지 처리한다. `--platform codex|cursor|claude-code|gemini-cli|kiro|copilot|copilot-cli` 등으로 특정 플랫폼만 지정 가능.
- 지원 플랫폼: Codex, Claude Code, Cursor, Windsurf, Zed, Continue, OpenCode, Antigravity, Gemini CLI, Qwen, Qoder, Kiro, GitHub Copilot(+CLI).
- Python 3.10+ 필요. [uv](https://docs.astral.sh/uv/)가 있으면 MCP 설정이 `uvx`를 사용한다.
- 500파일 프로젝트 기준 초기 빌드는 약 10초. 이후에는 워치 모드와 훅이 그래프를 자동 갱신한다.

## 언어 지원

- Python, JavaScript/TypeScript/TSX, Go, Rust, Java, C/C++, C#, Ruby, Kotlin, Swift, PHP, Scala, Solidity, Dart, R, Perl, Lua/Luau, Objective-C, 셸 스크립트, Elixir, Zig, PowerShell, Julia, ReScript, GDScript, Nix, Verilog/SystemVerilog, SQL, Vue/Svelte SFC, Astro, Jupyter/Databricks 노트북(`.ipynb`), Perl XS(`.xs`)까지 커버.
- **커스텀 언어 추가(포크 불필요)**: `.code-review-graph/languages.toml`에 확장자와 `tree_sitter_language_pack` 문법, 함수/클래스/임포트/호출 노드 타입만 매핑하면 제네릭 워커가 알아서 추출한다. 내장 언어는 오버라이드할 수 없다.

## GitHub Action (CI 리뷰)

- 같은 분석을 컴포지트 GitHub Action으로 실행할 수 있다. 그래프는 CI 러너 위에서만 빌드·쿼리되고 소스 코드는 외부로 전송되지 않는다.
- PR마다 위험도 점수가 매겨진 함수, 영향받는 실행 흐름, 테스트 공백을 담은 스티키 코멘트 하나를 게시하고 푸시마다 갱신한다. `fail-on-risk` 입력으로 머지 게이트로도 쓸 수 있다.

## 벤치마크 (정직한 수치 공개가 인상적)

- **헤드라인**: 6개 저장소에서 질문당 토큰 절감 중앙값 **~82배**(전체 코퍼스 기준 대비). 자주 인용되는 **528배는 최대치**(fastapi 단일 최고 사례)이지 일반적 결과가 아니라고 명시.
- 절감 범위 38배~528배: fastapi 528.4x(951,071→2,169 토큰), code-review-graph 93.0x, gin 91.8x, flask 71.4x, express 40.6x, httpx 38.0x. 일반적인 에이전트 질문에 그래프는 ~2,000–3,500토큰의 타깃 결과를 반환한다.
- 재현성: 13개 커밋, 업스트림 SHA 고정, Leiden 커뮤니티 감지 시드 고정, CPU 결정적 임베딩 — 다른 머신에서도 동일한 숫자가 나온다.
- **영향 정확도**: 평균 F1 0.71 (precision 0.578, recall 1.0). 단 recall 1.0은 그래프에서 유도한 정답을 같은 그래프로 예측하는 **순환 구조의 상한선**이지 "100% 재현율"이 아니라고 스스로 경고한다. git 이력 기반의 정직한 co-change 모드도 병행 측정 중.
- 토큰 추정치는 OpenAI `cl100k_base` 토크나이저와 교차 검증(`--verify`) 시 222개 샘플 파일 집계 기준 실제 GPT-4 토큰과 ~1% 이내 오차.
- **한계도 명시**: 작은 단일 파일 변경에서는 그래프 컨텍스트가 오히려 더 클 수 있음, 검색 품질 MRR 0.35, 플로 감지 재현율 33%(Python 프레임워크 위주), 영향 분석은 의도적으로 보수적(과예측 감수).

## 사용법 요약

- 슬래시 커맨드: `/code-review-graph:build-graph`, `/code-review-graph:review-delta`, `/code-review-graph:review-pr`.
- 주요 CLI: `build`, `update`, `status`, `watch`, `visualize`(HTML/GraphML/SVG/Obsidian/Neo4j Cypher), `wiki`, `detect-changes --brief`, `serve`(MCP 서버), `eval` 등.
- **Token Savings 패널**: `detect-changes --brief`(읽기 전용, ~1초)와 `update --brief`(그래프 갱신 후, ~5초)가 동일한 절감 패널을 출력. 예: 전체 컨텍스트 12,921토큰 → 그래프 762토큰(~94% 절감).
- **멀티 레포 데몬**: `crg-daemon add/start/status/logs/stop`으로 여러 저장소를 백그라운드에서 감시·자동 갱신. 설정은 `~/.code-review-graph/watch.toml`, 30초마다 헬스체크로 죽은 워처를 재시작.
- **MCP 도구 30종**: `get_minimal_context_tool`(~100토큰, 최우선 호출), `get_impact_radius_tool`, `get_review_context_tool`, `query_graph_tool`, `semantic_search_nodes_tool`, `detect_changes_tool`, 허브/브리지 감지, 지식 공백 분석, 리팩터링, 위키 생성, 크로스 레포 검색 등. 토큰이 빠듯한 환경에선 `--tools` 또는 `CRG_TOOLS`로 서브셋만 노출 가능.
- MCP 프롬프트 5종: `review_changes`, `architecture_map`, `debug_issue`, `onboard_developer`, `pre_merge_check`.
- 임베딩은 선택 사항: sentence-transformers 로컬, Google Gemini, MiniMax, 또는 OpenAI 호환 엔드포인트(Azure, LiteLLM, vLLM, LocalAI 등). 현재는 함수 시그니처만 임베딩(~10토큰/노드)하므로 장문 컨텍스트 강점 모델의 이점이 제한적이라는 점도 밝힌다.
- 인덱싱 제외는 `.code-review-graphignore` 파일로. git 저장소에서는 추적 파일만 인덱싱된다(`git ls-files`).

## 다른 접근과의 비교 (FAQ)

- **vs LSP**: 언어별 데몬 대신 하나의 영속적 크로스 언어 그래프. 심볼 단위 정밀도는 LSP가 우위.
- **vs RAG/임베딩**: 유사도 청크가 아닌 AST에서 파싱한 구조적 엣지. 임베딩은 검색 보조용 옵션일 뿐.
- **vs grep**: 원홉 조회는 grep이 이기고, 멀티홉 질문(영향 반경, 호출자의 호출자, 테스트 매핑)은 그래프가 이긴다.
- 작은 저장소, 사소한 단일 파일 diff, 일회성 질문에는 쓰지 말라고 스스로 권한다. 텔레메트리 없음, 클라우드 임베딩은 옵트인.

## 트러블슈팅 메모

- pip/pipx가 PyPI에서 hatchling을 못 받는 경우: IDE 내장 터미널 대신 Terminal.app에서 재시도하거나 `uv tool install . --force` 사용.
- Windows에서 `Invalid JSON: EOF while parsing` / `MCP error -32000` 발생 시: `cmd /c` 래퍼를 쓰지 말고 `.exe`를 직접 실행하며 `PYTHONUTF8=1` 환경변수를 설정, `fastmcp` 3.2.4+ 필요.
- 라이선스: MIT.

## 관련 노트

- [[zilliz-claude-context]] — AI 코딩용 시맨틱 코드검색 MCP(비슷한 컨텍스트 절감 접근)
- [[harness-engineering-codex]] — 린터·구조 테스트로 에이전트 가독성을 기계적으로 강제

> 원문: [tirth8205/code-review-graph: Local-first code intelligence graph for MCP and CLI](https://github.com/tirth8205/code-review-graph)
> 원본 클립: [[2026-07-14-tirth8205code-review-graph Local-first code intelligence graph for MCP and CLI. Builds a persistent map of your codebase so AI coding tools read only what matters, with benchmarked context reductions on reviews and large-repo workflow]]
