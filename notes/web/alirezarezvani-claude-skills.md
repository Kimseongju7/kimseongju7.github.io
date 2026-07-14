---
title: "alirezarezvani/claude-skills — 13개 코딩 에이전트용 345+ 스킬·플러그인 라이브러리"
date: 2026-07-14 14:14:13 +0900
categories: [개발, 도구]
tags: [claude-code, skills, plugins, agents, codex, gemini-cli, cursor, open-source]
slug: alirezarezvani-claude-skills
publish: true
---
alirezarezvani/claude-skills는 355개의 프로덕션급 Claude Code 스킬·플러그인·에이전트 스킬을 모은 최대 규모의 오픈소스 라이브러리다. Claude Code뿐 아니라 OpenAI Codex, Gemini CLI, Cursor 등 13개 AI 코딩 도구에서 쓸 수 있으며, GitHub 스타 5,200+를 기록하고 있다.

## 스킬이란 무엇인가

Claude Code 스킬(에이전트 스킬, 코딩 에이전트 플러그인)은 AI 코딩 에이전트에게 기본기에 없는 도메인 전문성을 주는 모듈형 지침 패키지다. 각 스킬은 다음으로 구성된다.

- **SKILL.md** — 구조화된 지침, 워크플로, 의사결정 프레임워크
- **Python 도구** — 602개 CLI 스크립트(전부 표준 라이브러리만 사용, pip 설치 불필요)
- **참조 문서** — 711개의 템플릿, 체크리스트, 도메인 지식 파일

스킬(어떻게 실행하나) / 에이전트(무슨 작업을 하나) / 페르소나(누가 생각하나)의 세 층위가 함께 동작한다. 예: 스킬은 "SEO는 이 단계대로", 에이전트는 "보안 감사를 실행", 페르소나는 "스타트업 CTO처럼 사고".

## 설치

- **Claude Code**: `/plugin marketplace add alirezarezvani/claude-skills` 후 도메인별 번들 설치 — 예: `engineering-skills`(핵심 엔지니어링 24개), `engineering-advanced-skills`(POWERFUL 티어 25개), `marketing-skills`(43개), `c-level-skills`(28개) 등. `skill-security-auditor`, `playwright-pro`, `self-improving-agent`, `content-creator` 같은 개별 스킬 설치도 가능.
- **Gemini CLI**: 레포 클론 후 `./scripts/gemini-install.sh` 실행, `activate_skill(name="senior-architect")`로 사용
- **OpenAI Codex**: `npx agent-skills-cli add alirezarezvani/claude-skills --agent codex` 또는 `./scripts/codex-install.sh`
- **수동 설치**: 스킬 폴더를 `~/.claude/skills/`(Claude Code) 또는 `~/.codex/skills/`(Codex)에 복사

## 멀티 툴 지원

`./scripts/convert.sh --tool all` 스크립트 하나로 345개 스킬 전부를 9개 코딩 도구의 네이티브 포맷으로 변환한다(약 15초). Cursor는 `.mdc` 룰, Aider는 `CONVENTIONS.md`, Kilo Code / Windsurf / OpenCode / Augment / Antigravity는 각자의 규칙·스킬 폴더로 설치된다. Hermes Agent와 Mistral Vibe는 agentskills.io의 SKILL.md 표준을 그대로 쓰므로 포맷 변환 없이 동기화 스크립트만 실행하면 된다.

## 도메인 구성 (18개 도메인, 355개 스킬)

- **엔지니어링 — 핵심 (52)**: 아키텍처, 프론트/백엔드/풀스택, QA, DevOps, SecOps, AI/ML, 데이터, Playwright Pro, 자기개선 에이전트(auto-memory curation), 보안 스위트, 접근성 감사, 유명 엔지니어링 철학 기반의 적대적 리뷰(named-persona-adversarial-review)
- **엔지니어링 — POWERFUL (81)**: 에이전트 디자이너, RAG 아키텍트, DB 디자이너, CI/CD 빌더, 보안 감사, MCP 빌더, Helm/Terraform, 신뢰성 포트폴리오(feature flags, Kubernetes operator, 카오스 엔지니어링, SLO), [[mattpocock-skills|Matt Pocock 스킬]], zero-hallucination-coder(Discuss→Map→Decompose→Execute→Verify), agent-harness 등
- **프로덕트 (17)**: PM, 애자일 PO, UX 리서처, UI 디자인, 랜딩 페이지, SaaS 스캐폴더, code-to-prd, apple-hig-expert 등
- **마케팅 (48)**: 콘텐츠, SEO + AEO(Answer Engine Optimization — LLM 인용 최적화, 5개 LLM에 걸친 인용 추적), 로컬 SEO, CRO, 채널, 그로스, 인텔리전스, 세일즈의 8개 포드
- **생산성 (7)**: capture(브레인덤프→액션), email 페어, reflect(저널), handoff, andreessen, roast(5각도 아이디어 패널 → GO/RESHAPE/KILL)
- **학술 연구 (9)**: research 오케스트레이터 + litreview, grants(NIH), dossier, patent, syllabus, notebooklm, deep-research 등 8개 전문 스킬
- **Research Operations (5, v2.9.0)**: 엔터프라이즈용 — clinical-research, research-finance, market-research, product-research
- 그 외 프로젝트 관리(9), 규제·품질(19 — ISO 13505, MDR 2017/745, FDA, ISO 27001, GDPR, SOC 2, CAPA), Compliance OS(9), C-레벨 자문(68 — CEO~VPE 풀 C-suite + 파운더 모드 + 21개 /cs:* 슬래시 커맨드), 비즈니스 & 그로스(5), 비즈니스 운영(7), 커머셜(8), 파이낸스(4), Loop Library(1), Markdown→HTML(5)

## 페르소나와 오케스트레이션

큐레이션된 스킬 로드아웃과 소통 스타일을 가진 사전 구성 에이전트 정체성으로 **Startup CTO**(아키텍처 결정, 기술 스택 선정), **Growth Marketer**(콘텐츠 주도 성장, 론칭 전략), **Solo Founder**(1인 스타트업, MVP)가 있다.

프레임워크 없이 페르소나·스킬·에이전트를 조율하는 경량 오케스트레이션 프로토콜은 4가지 패턴을 제시한다.

| 패턴 | 내용 | 적합한 상황 |
| --- | --- | --- |
| Solo Sprint | 프로젝트 단계별로 페르소나 전환 | 사이드 프로젝트, MVP, 1인 창업 |
| Domain Deep-Dive | 페르소나 1개 + 스킬 여러 개 중첩 | 아키텍처 리뷰, 컴플라이언스 감사 |
| Multi-Agent Handoff | 페르소나들이 서로의 결과물을 리뷰 | 고위험 결정, 론칭 준비 점검 |
| Skill Chain | 페르소나 없이 스킬 순차 실행 | 콘텐츠 파이프라인, 반복 체크리스트 |

## 보안 감사 도구

v2.0.0에 추가된 skill-security-auditor로 설치 전에 어떤 스킬이든 보안 위험을 감사할 수 있다.

```
python3 engineering/skill-security-auditor/scripts/skill_security_auditor.py /path/to/skill/
```

커맨드 인젝션, 코드 실행, 데이터 유출, 프롬프트 인젝션, 의존성 공급망 위험, 권한 상승을 스캔하고 PASS / WARN / FAIL과 개선 가이드를 반환한다. 의존성 없이 Python만 있으면 동작한다.

## 기타

- 스킬과 함께 제공되는 580개 CLI 분석 도구: SaaS 지표 계산, 브랜드 보이스 분석, 기술부채 스코어링, RICE 우선순위화, 랜딩 페이지 스캐폴딩 등
- 시맨틱 버저닝을 따르며 패치 릴리스 내 하위 호환을 유지한다(스크립트 인자, 플러그인 경로, SKILL.md 구조 불변)
- 자기만의 스킬 제작: 스킬은 `SKILL.md`(frontmatter + 지침) + 선택적 `scripts/`, `references/`, `assets/` 폴더 — 단계별 가이드는 별도 레포 Skills & Agents Factory 참고
- 관련 프로젝트: Claude Code Skills & Agents Factory(스킬 대량 제작 방법론), Claude Code Tresor(프롬프트 템플릿 60+), Product Manager Skills, toprank(SEO·Google Ads 스킬 9개)
- 라이선스: MIT

## 관련 노트

- [[mattpocock-skills]] — 라이브러리에도 포함된 Matt Pocock 스킬의 원 출처
- [[midudev-autoskills]] — 스킬 자동 생성 접근
- [[codex-top-6-skills]] — Codex용 대표 스킬 모음
- [[prompt-master-skill]] — 프롬프트 작성 스킬
- [[interface-design-claude-code]] — UI 디자인 엔지니어링 스킬

> 원문: [alirezarezvani/claude-skills: 345 Claude Code skills & agent skills & plugins](https://github.com/alirezarezvani/claude-skills)
> 원본 클립: [[2026-07-14-alirezarezvaniclaude-skills 345 Claude Code skills & agent skills & plugins (30+ Agents, 70+ custom commands, 330+ skills, customizable references, scripts)for Claude Code, Codex, Gemini CLI, Cursor, and 8 more coding agents — enginee]]
