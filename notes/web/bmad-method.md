---
title: "BMAD-METHOD — 애자일 AI 주도 개발(Agile AI Driven Development) 프레임워크"
date: 2026-07-14 14:35:43 +0900
categories: [개발, 도구]
tags: [bmad, agile, ai-agents, workflow, claude-code, open-source]
slug: bmad-method
publish: true
---
**BMAD-METHOD**(Build More Architect Dreams)는 버그 수정부터 엔터프라이즈 시스템까지 프로젝트 규모에 맞춰 계획 깊이를 조절하는 스케일 적응형(scale-adaptive) 애자일 AI 주도 개발 프레임워크다. 100% 무료 오픈소스(MIT 라이선스)로, 유료 장벽·게이트된 콘텐츠·유료 Discord가 전혀 없다.

## 왜 BMad Method인가

전통적인 AI 도구는 생각을 대신해 주기 때문에 평균적인 결과물을 낸다. 반면 BMad의 에이전트와 진행형 워크플로는 전문가 협업자처럼 구조화된 프로세스로 사용자를 이끌어, AI와의 파트너십 속에서 사용자 최고의 사고를 끌어내는 것을 목표로 한다.

- **AI 지능형 도움말** — 언제든 `bmad-help` 스킬을 호출하면 다음에 할 일을 안내받을 수 있다 (예: `bmad-help I just finished the architecture, what do I do next?`)
- **규모·도메인 적응형** — 프로젝트 복잡도에 따라 계획 깊이를 자동 조절
- **구조화된 워크플로** — 분석·계획·아키텍처·구현 전반에 걸친 애자일 모범 사례 기반
- **전문 에이전트** — PM, 아키텍트, 개발자, UX 등 12개 이상의 도메인 전문가
- **파티 모드(Party Mode)** — 여러 에이전트 페르소나를 한 세션에 불러 함께 토론·협업
- **전체 라이프사이클** — 브레인스토밍부터 배포까지 커버

## 로드맵 (V6)

V6가 출시됐고 이제 시작이다. Cross Platform Agent Team과 Sub Agent 통합, Skills Architecture, BMad Builder v1, Dev Loop Automation 등 최적화가 빠르게 진행 중이다. 전체 로드맵은 [docs.bmad-method.org/roadmap](https://docs.bmad-method.org/roadmap/)에서 확인할 수 있다.

## 빠른 시작

사전 요구사항: Node.js v20.12+, Python 3.10+, uv

```
npx bmad-method install
```

최신 프리릴리스 빌드는 `npx bmad-method@next install` (기본 설치보다 변동이 잦음). 설치 프롬프트를 따른 뒤 프로젝트 폴더에서 AI IDE(Claude Code, Cursor 등)를 열면 된다.

CI/CD용 비대화형 설치:

```
npx bmad-method install --directory /path/to/project --modules bmm --tools claude-code --yes
```

모듈 설정은 `--set <module>.<key>=<value>`(반복 가능)로 덮어쓸 수 있고, `--list-options [module]`로 로컬에 알려진 공식 키를 조회한다:

```
npx bmad-method install --yes \
  --modules bmm --tools claude-code \
  --set bmm.project_knowledge=research \
  --set bmm.user_skill_level=expert
```

## 모듈 생태계

BMad Method는 전문 도메인용 공식 모듈로 확장된다. 설치 중이나 이후 언제든 추가할 수 있다.

| 모듈 | 용도 |
| --- | --- |
| **BMad Method (BMM)** | 34개 이상의 워크플로를 담은 코어 프레임워크 |
| **BMad Builder (BMB)** | 커스텀 BMad 에이전트·워크플로 제작 |
| **Test Architect (TEA)** | 리스크 기반 테스트 전략과 자동화 |
| **Game Dev Studio (BMGD)** | 게임 개발 워크플로 (Unity, Unreal, Godot) |
| **Creative Intelligence Suite (CIS)** | 혁신, 브레인스토밍, 디자인 씽킹 |

## 웹 번들 (Web Bundles)

V4에서 처음 나왔던 웹 번들이 V6에서 개선되어 돌아왔다. 웹 번들은 선별한 BMad 스킬을 **Google Gemini Gems**와 **ChatGPT Custom GPT**로 설치할 수 있게 패키징한 것이다. 브레인스토밍, 제품 브리프, PRD, PRFAQ, UX 스펙, 시장·산업 조사 같은 사전 계획 작업을 웹 LLM 구독으로 처리한 뒤, 다듬어진 산출물을 IDE로 가져와 구현하는 방식이다. 계획 단계가 종량제 IDE 토큰 대신 정액 구독으로 돌아가므로 긴 프로젝트에서 의미 있는 비용 절감이 된다.

현재 제공: 브레인스토밍, 제품 브리프, PRFAQ, PRD, UX, 시장·산업 조사. [bmadcode.com/web-bundles](https://bmadcode.com/web-bundles/)에서 번들별 카드, Gemini/ChatGPT 인라인 설치 안내, 원클릭 ZIP 다운로드를 제공한다.

## 문서·커뮤니티·라이선스

- 문서: [docs.bmad-method.org](https://docs.bmad-method.org/) — 튜토리얼, 가이드, 개념, 레퍼런스 (시작 튜토리얼, V6 업그레이드 가이드, Test Architect 문서 포함)
- 커뮤니티: Discord, YouTube(@BMadCode), X/Twitter, GitHub Issues/Discussions
- 후원: 저장소 스타, buy me a coffee, 기업 스폰서십은 contact@bmadcode.com
- 라이선스: MIT. **BMad**와 **BMAD-METHOD**는 BMad Code, LLC의 상표다.

## 관련 노트

- [[ouroboros-agent-os]] — 스펙 우선 에이전트 개발 프레임워크(BMad와 유사한 워크플로 주도)
- [[ouroboros-readme-ko]] — Ouroboros Agent OS 한국어 소개
- [[harness-engineering-codex]] — 에이전트 우선 개발에서 환경·워크플로를 설계하는 사례

> 원문: [bmad-code-org/BMAD-METHOD: Breakthrough Method for Agile Ai Driven Development](https://github.com/bmad-code-org/bmad-method)
> 원본 클립: [[2026-07-14-bmad-code-orgBMAD-METHOD Breakthrough Method for Agile Ai Driven Development]]
