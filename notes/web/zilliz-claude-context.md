---
title: "zilliztech/claude-context — 코드베이스 전체를 컨텍스트로 만드는 시맨틱 코드 검색 MCP"
date: 2026-07-14 14:10:35 +0900
categories: [개발, 도구]
tags: [mcp, claude-code, semantic-search, code-search, vector-database, milvus, zilliz, embedding]
slug: zilliz-claude-context
publish: true
---
**Claude Context**는 Claude Code를 비롯한 AI 코딩 에이전트에 시맨틱 코드 검색을 붙여 주는 MCP 플러그인이다. 코드베이스 전체를 벡터 데이터베이스에 인덱싱해 두고 질의와 관련된 코드만 컨텍스트로 가져오므로, 수백만 라인 규모에서도 여러 번의 탐색 없이 관련 코드를 바로 찾고 비용도 절감할 수 있다.

## 개요

- **코드베이스 전체를 컨텍스트로**: 시맨틱 검색으로 수백만 라인에서 관련 코드를 한 번에 찾아 Claude의 컨텍스트로 직접 가져온다. 여러 라운드에 걸친 파일 탐색(discovery)이 필요 없다.
- **대규모 코드베이스에서 비용 효율적**: 요청마다 디렉터리 전체를 로드하는 대신, 코드베이스를 벡터 DB에 저장해 두고 관련 코드만 컨텍스트에 사용한다.
- 같은 팀에서 만든 세션 간 장기 기억(persistent memory) 플러그인으로 [memsearch](https://github.com/zilliztech/memsearch#for-claude-code-users)도 있다(마크다운 기반 메모리 시스템).

## 빠른 시작

### 사전 준비물

- **벡터 데이터베이스**: [Zilliz Cloud](https://cloud.zilliz.com/signup)에 가입해 무료 벡터 DB와 API 키(Personal Key)를 받는다.
- **임베딩 모델용 OpenAI API 키**: [OpenAI](https://platform.openai.com/api-keys)에서 발급받는다(`sk-`로 시작).
- **시스템 요구사항**: Node.js >= 20.0.0

### Claude Code에 MCP 등록

```
claude mcp add claude-context \
  -e OPENAI_API_KEY=sk-your-openai-api-key \
  -e MILVUS_ADDRESS=your-zilliz-cloud-public-endpoint \
  -e MILVUS_TOKEN=your-zilliz-cloud-api-key \
  -- npx @zilliz/claude-context-mcp@latest
```

### 사용 흐름

1. 프로젝트 디렉터리에서 `claude` 실행
2. `Index this codebase` — 코드베이스 인덱싱
3. `Check the indexing status` — 인덱싱 상태 확인
4. `Find functions that handle user authentication` 같은 자연어 검색

## 다른 MCP 클라이언트 지원

stdio 전송과 표준 MCP 프로토콜을 따르므로 `npx @zilliz/claude-context-mcp@latest` 명령으로 어떤 MCP 호환 클라이언트와도 연동된다. 문서에 설정 예시가 제공되는 클라이언트:

- OpenAI Codex CLI (`~/.codex/config.toml`, 최상위 키가 `mcpServers`가 아니라 `mcp_servers`인 점 주의, `startup_timeout_ms`로 기본 10초 타임아웃 조정 가능)
- Gemini CLI (`~/.gemini/settings.json`), Qwen Code (`~/.qwen/settings.json`)
- Cursor (`~/.cursor/mcp.json`), Void, Claude Desktop, Windsurf, VS Code
- Cherry Studio(GUI 설정), Cline, Augment(UI 또는 `augment.advanced` 수동 설정), Roo Code, Zencoder
- LangChain/LangGraph 연동 예시도 저장소에 있다.

공통 환경 변수는 `OPENAI_API_KEY`, `MILVUS_ADDRESS`(Zilliz Cloud 퍼블릭 엔드포인트), `MILVUS_TOKEN`(Zilliz Cloud API 키)이다.

## 제공 도구 (MCP Tools)

1. `index_codebase` — 하이브리드 검색(BM25 + dense vector)용 코드베이스 인덱싱
2. `search_code` — 자연어 질의로 인덱싱된 코드베이스를 하이브리드 검색
3. `clear_index` — 특정 코드베이스의 검색 인덱스 삭제
4. `get_indexing_status` — 인덱싱 진행률/완료 상태 조회

## 평가 결과

자체 통제 실험에서 Claude Context MCP는 **동등한 검색 품질 조건에서 약 40%의 토큰 절감**을 달성했다. 프로덕션 환경에서의 비용·시간 절감으로 이어지며, 제한된 컨텍스트 길이 안에서는 더 나은 검색·답변 품질을 의미한다.

## 아키텍처와 구현

- **하이브리드 코드 검색**: BM25 + dense vector 결합
- **증분 인덱싱**: Merkle tree로 변경된 파일만 효율적으로 재인덱싱
- **지능형 코드 청킹**: AST(추상 구문 트리) 기반 분할(실패 시 LangChain 문자 기반 분할로 폴백)
- **확장성**: Zilliz Cloud 연동으로 코드베이스 크기와 무관하게 확장
- **커스터마이즈**: 파일 확장자, ignore 패턴, 임베딩 모델 설정 가능

### 모노레포 구성 패키지

- `@zilliz/claude-context-core` — 임베딩·벡터 DB 통합을 담은 핵심 인덱싱 엔진
- **VSCode Extension** — "Semantic Code Search" 확장 (마켓플레이스에서 설치 가능)
- `@zilliz/claude-context-mcp` — AI 에이전트 연동용 MCP 서버

### 지원 기술

- 임베딩 제공자: OpenAI, VoyageAI, Ollama, Gemini (커스텀 모델 예: `text-embedding-3-large`, `voyage-code-3`)
- 벡터 DB: Milvus 또는 Zilliz Cloud(완전 관리형)
- 지원 언어: TypeScript, JavaScript, Python, Java, C++, C#, Go, Rust, PHP, Ruby, Swift, Kotlin, Scala, Markdown

## MCP 외의 사용 방법

- **Core 패키지로 앱 구축**: `@zilliz/claude-context-core`의 `Context`, `MilvusVectorDatabase`, `OpenAIEmbedding`으로 직접 인덱싱(`indexCodebase`, 진행률 콜백 지원)과 시맨틱 검색(`semanticSearch`)을 수행할 수 있다.
- **VSCode 확장**: IDE 안에서 시맨틱 코드 검색·탐색 UI 제공.

## 개발 환경

- Node.js 20.x/22.x/24.x, pnpm 권장. `pnpm install` → `pnpm build` → `pnpm dev`.
- 패키지별 빌드(`pnpm build:core / build:vscode / build:mcp`)와 `pnpm benchmark` 지원. Windows는 rimraf 기반 크로스 플랫폼 스크립트로 동작하며 `git config core.autocrlf false` 권장.

## 로드맵과 라이선스

- 로드맵: AST 기반 코드 분석 고도화, 임베딩 제공자 추가, 에이전트 기반 대화형 검색 모드, 청킹 전략 개선, 검색 결과 랭킹 최적화, Chrome 확장 등
- 라이선스: MIT

## 관련 노트

- [[code-review-graph]] — 코드베이스를 지능형 그래프로 색인해 탐색한다는 유사한 코드 인텔리전스 접근
- [[headroom-context-compression]] — 관련 코드만 컨텍스트로 가져와 토큰을 아끼는, 컨텍스트 절약과 맞닿은 주제
- [[openai-codex-plugin-cc]] — 이 MCP 서버를 Codex CLI(`mcp_servers`)에 붙일 때 참고할 Codex 플러그인

> 원문: [zilliztech/claude-context: Code search MCP for Claude Code. Make entire codebase the context for any coding agent.](https://github.com/zilliztech/claude-context)
> 원본 클립: [[2026-07-14-zilliztechclaude-context Code search MCP for Claude Code. Make entire codebase the context for any coding agent.]]
