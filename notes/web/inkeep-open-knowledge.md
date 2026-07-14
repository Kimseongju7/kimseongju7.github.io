---
title: "inkeep/open-knowledge — AI 네이티브 마크다운 IDE이자 LLM 위키, OpenKnowledge"
date: 2026-07-14 14:11:35 +0900
categories: [개발, 도구]
tags: [markdown, wysiwyg, llm, wiki, knowledge-base, mcp, obsidian, note-taking]
slug: inkeep-open-knowledge
publish: true
---
**OpenKnowledge**는 Claude, Codex 등 여러 하네스와 통합되는 아름다운 마크다운 에디터다. 지식 베이스, LLM 위키, 스펙, 노트를 위한 도구이며 프라이빗하고 로컬에서 동작하며 무료다. "Notion과 VS Code의 만남"으로 생각하면 된다.

## 주요 특징

- **완전한 WYSIWYG**: 마크다운 파일 편집이 Google Doc이나 Notion 페이지를 편집하는 느낌.
- **macOS 앱 + 웹 UI**: 파일 내비게이터, 검색, 탭, 그래프 위키링크 뷰어 등 제공.
- **사이드바이사이드 AI 편집 통합**: Claude, Codex, OpenCode, Pi 등과 함께 편집. MCP/CLI를 통해 어떤 하네스/에이전트와도 사용 가능.
- **MCP·스킬·에이전틱 검색 기본 제공**: LLM 위키, 세컨드 브레인, 지식 그래프용.
- **노코드 팀 공유와 자동 동기화**: 내부적으로 git/GitHub 기반.
- **임베더블 HTML과 리치 컴포넌트**: 엔지니어링 스펙과 시각화 리포트 작성용.

## 설치

- **macOS**: 데스크톱 앱 다운로드 — DMG를 열어 OpenKnowledge를 Applications로 드래그 후 실행. [최신 릴리스](https://github.com/inkeep/open-knowledge/releases/latest)
- **Linux, Windows, Intel Mac**: CLI로 같은 에디터를 로컬 웹 앱으로 실행 (Node.js 24+ 와 git 필요):

```
npm install -g @inkeep/open-knowledge
cd your-project
ok init          # 프로젝트 스캐폴딩 + AI 에디터 연동 (Claude Code, Claude Desktop, Cursor, Codex, OpenCode, OpenClaw)
ok start --open  # 웹 에디터를 서빙하고 브라우저에서 열기
```

## 사용법

- 마크다운/mdx 파일이 있는 컴퓨터의 아무 폴더나 열어서 사용한다. 기존 코드베이스, 위키, **Obsidian 볼트** 등과 함께 쓸 수 있다.
- 처음엔 단순히 리치 WYSIWYG 마크다운 에디터로 시작해도 되고, 더 나아가려면 스타터 팩으로 LLM 위키, 세컨드 브레인, 구조화된 지식 베이스를 만들 수 있다.
- 어느 쪽이든 앱이 컴퓨터에서 감지된 에이전트 하네스용 MCP와 스킬 설치를 안내한다. 이들은 에이전트의 문서 검색·저작을 강화하도록 설계됐다.
- git/GitHub 기반 동기화와 공유는 선택적으로 활성화할 수 있다.
- 일반 사용 문서: [https://openknowledge.ai/docs](https://openknowledge.ai/docs)

## 라이선스와 커뮤니티

- 라이선스: GNU GPL v3.0 or later (OSI 승인 오픈소스 라이선스)
- 퍼블릭 PR/이슈 환영 (CONTRIBUTING.md 참고), 질문은 Discord 커뮤니티에서 가능하며 X([@OpenKnowledge](https://x.com/OpenKnowledge))에서 제품 업데이트를 알린다.

## 관련 노트

- [[harness-engineering-codex]] — 에이전트가 읽는 리포지터리 지식 기록 시스템과 통하는 문제의식
- [[code-review-graph]] — 로컬 MCP 코드 지능 그래프(Obsidian 내보내기 지원)

> 원문: [inkeep/open-knowledge: Beautiful, AI-native markdown IDE and LLM wiki](https://github.com/inkeep/open-knowledge)
> 원본 클립: [[2026-07-14-inkeepopen-knowledge Beautiful, AI-native markdown IDE and LLM wiki]]
