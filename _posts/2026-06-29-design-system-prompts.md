---
title: "디자인 시스템 프롬프트 모음"
date: 2026-06-29 13:39:38 +0900
categories: [메모]
tags: [prompt, design-system, ai]
description: "인터넷에서 주운 디자인 시스템 관련 프롬프트 모음 — 오케스트레이션형, DESIGN.md 팁, 역할 부여형 단발 프롬프트."
---

인터넷에서 주운 디자인 시스템 관련 프롬프트 모음. 출처는 [2026-06-29](/posts/daily-2026-06-29/) 일지에서 분리.

## 한눈에

- **1~2번** — "끝까지 자율적으로 완수하라"는 동일한 [오케스트레이션 지시문](#공통-오케스트레이션-지시문-1-2번)을 공유하고, 앞부분의 Task만 다름.
- **3번** — 프롬프트라기보다 팁. 평문 디자인 문서(DESIGN.md)를 두는 방법.
- **4~8번** — 역할 부여형(Act as ...) 단발 프롬프트.

---

## 번역 및 설명

### 1. 디자인 시스템 — 흩어진 UI를 컴포넌트 라이브러리로 통합

**용도**: 프로젝트 곳곳에 중복·일회성으로 만들어진 UI 컴포넌트, 스타일 토큰, 반복 인터랙션을 하나의 재사용 가능한 디자인 시스템으로 리팩토링하는 대규모 자동화 작업.

**Task 번역**:
- 흩어진 UI 컴포넌트·스타일 토큰·반복 인터랙션을 재사용 가능하고 유지보수하기 좋은 하나의 통합 디자인 시스템(컴포넌트 라이브러리)으로 추상화하라.
- 직접 만든 일회성 컴포넌트를 라이브러리 컴포넌트로 교체하고, 모든 새 UI는 우회 없이 라이브러리를 거치게 하라.
- 마지막에 리뷰 스킬을 추가하라.
- 리팩토링이므로 worktree에서 시작하라.

→ 이후는 [공통 오케스트레이션 지시문](#공통-오케스트레이션-지시문-1-2번) 참고.

### 2. 접근성 + 멀티 디바이스

**용도**: 일반/저시력/키보드 전용/다양한 기기 사용자가 모두 핵심 동작을 안정적으로 수행하도록 반응형·대비·키보드·포커스·a11y 시맨틱을 점검하고 고치는 자동화 작업.

**Task 번역**:
- 일반·저시력·키보드 전용·서로 다른 기기 사용자가 모두 핵심 동작을 안정적으로 완료할 수 있게 하라.
- 현재 커밋에서 앱을 로컬로 띄우고, 데스크톱/태블릿/폰 뷰포트에서 핵심 경로를 돌아보라.
- 다음을 고쳐라 — 반응형 문제(겹침·넘침·잘림·가로 스크롤), 대비, 폰트 스케일, 키보드(Tab/Enter/Space/Esc), 포커스 상태, 이미지·아이콘 버튼·폼·상태의 a11y 시맨틱.
- 발견한 문제·수정 내용·남은 위험을 기록하고, 마지막에 리뷰 스킬을 추가하라.

→ 이후는 [공통 오케스트레이션 지시문](#공통-오케스트레이션-지시문-1-2번) 참고.

### 3. 먼저 DESIGN.md를 줘라

**용도**: 에이전트가 UI를 만들기 전 매번 읽는 평문 디자인 시스템 문서(DESIGN.md)를 두어 스타일이 흔들리지 않게 하는 팁.

**번역**:
- DESIGN.md는 Google Stitch에서 나온 개념으로, 색·타입 스케일·간격·반경·그림자·컴포넌트 규칙을 명시한 평문 디자인 시스템이다.
- 에이전트가 UI를 만들 때마다 이걸 먼저 읽어 스타일이 흔들리지 않게 한다.
- AGENTS.md가 "어떻게 만드는가"라면, DESIGN.md는 "어떻게 보여야 하는가"다.
- 직접 쓰기 싫다면 가져다 써라 — 누군가 Stripe·Linear·Airbnb·Vercel 등 실제 73개를 [awesome-design-md](https://github.com/VoltAgent/awesome-design-md)에 모아뒀다.
- 루트에 하나 넣고 모든 UI가 DESIGN.md를 따르게 시켜라.

### 4. UX 감사 — 먼저 현재 상태를 보게 하라

**용도**: 시니어 UX 디자이너 역할로 핵심 플로우를 따라가며 문제점을 심각도별로 진단하고 구체적 수정안을 받는 점검용 프롬프트.

**번역**: 시니어 UX 디자이너로서 이 앱의 핵심 플로우를 감사하라. 실제 사용자 경로를 처음부터 끝까지 따라가며 약한 정보 위계, 마찰, 누락된 피드백, 오류 유발 단계, 만들어지지 않은 빈/오류 상태를 찾아라. 각 항목을 심각도로 평가하고 구체적인 수정안을 제시하라.

### 5. 한 페이지를 진짜 감각으로 다시 만들기

**용도**: 제품 디자이너 역할로 새 기능 추가 없이 한 페이지의 비주얼만 Linear/Stripe 수준으로 재구성.

**번역**: 제품 디자이너로서 이 페이지를 Linear/Stripe 같은 느낌으로 다시 만들어라. 먼저 스타일부터 확정하라 — 절제된 팔레트, 명확한 타입 스케일, 충분한 여백, 일관된 반경과 그림자. 그다음 만들어라. 비주얼만 건드리고 새 기능은 추가하지 말며, 모든 변경을 하나하나 설명하라.

### 6. 모든 인터랙션과 상태를 제대로

**용도**: 시니어 프론트엔드 엔지니어 역할로 로딩/빈/오류/hover·focus·disabled 등 모든 상태와 인터랙션을 빠짐없이 구현.

**번역**: 시니어 프론트엔드 엔지니어로서 모든 인터랙션과 상태를 제대로 처리하라. 로딩 스켈레톤, 빈 상태, 오류 상태, hover/focus/disabled, 그리고 핵심 동작의 트랜지션과 피드백을 모두 다뤄라. 재사용 가능하고 접근성 있게 만든 뒤 컴포넌트 구조와 전체 구현을 내놓아라.

### 7. 전환되는 랜딩 페이지 만들기

**용도**: 전환 중심 디자이너 역할로 가치 제안과 CTA를 잡고 완성도 있는 랜딩 페이지 제작.

**번역**: 전환을 중시하는 디자이너로서 랜딩 페이지를 만들어라. 먼저 한 줄 가치 제안과 방문자에게 원하는 단 하나의 행동을 못 박아라. 그다음 히어로, 사회적 증거, 기능 섹션, FAQ, CTA를 배치하라. 깔끔한 비주얼, 명확한 위계, 모바일 우선. 완성되어 바로 배포 가능한 페이지를 내놓아라.

### 8. 폼을 쓸 만하게 만들기

**용도**: 시니어 프론트엔드 엔지니어 역할로 폼의 검증·오류·기본값·키보드/자동완성·제출 상태 등을 챙겨 사용성 향상.

**번역**: 시니어 프론트엔드 엔지니어로서 이 폼을 쓸 만하게 만들어라. 실시간 검증, 명확한 인라인 오류, 합리적인 기본값, 키보드·자동완성 친화성, 제출 중 및 성공/실패 상태, 긴 폼의 단계 분할을 모두 다뤄라. 전체 구현을 내놓아라.

---

## 공통 오케스트레이션 지시문 (1·2번)

1·2번 프롬프트의 Task 뒤에 동일하게 붙는 "끝까지 자율적으로 완수하라"는 지시문:

1. **end-to-end /goal 작성** — 다음 단계만이 아니라 아키텍처·구현·테스트·리뷰·최종 결과가 기준을 만족할 때까지 전체 계획을 끝내는 새 /goal을 스스로 작성.
2. **분할 + 병렬 에이전트** — 목표를 독립적인 조각으로 나누고, 필요한 만큼 병렬 에이전트를 띄우되 각 에이전트에 기대 산출물·검증·완료 기준을 담은 전용 /goal 부여.
3. **동시 실행 + 추적** — 동시에 디스패치하고, 진행 상황을 적절한 곳에 계속 추적.
4. **종합 + 충돌 해소** — 결과가 돌아오는 대로 종합하고, 충돌을 해소하며, 구현을 이어감.
5. **실시간 검증** — 중요한 단계마다 검증을 돌리고, 리뷰·적절한 시점의 커밋·최종 요약으로 마무리. 검증은 브라우저/컴퓨터 사용, 클릭, 키보드 동작 등 실제 end-to-end 경로 포괄.
6. **멈추지 마라** — 자격증명 누락·파괴적 모호함·요구사항 충돌로 막히지 않는 한 부분적 진행에서 멈추지 마라.

---

## 원문 프롬프트

<details>
<summary>영어 원문 펼치기</summary>

1. 𝗗𝗲𝘀𝗶𝗴𝗻 𝗦𝘆𝘀𝘁𝗲𝗺 (𝗽𝘂𝗹𝗹 𝘀𝗰𝗮𝘁𝘁𝗲𝗿𝗲𝗱 𝗨𝗜 𝗶𝗻𝘁𝗼 𝗮 𝗰𝗼𝗺𝗽𝗼𝗻𝗲𝗻𝘁 𝗹𝗶𝗯𝗿𝗮𝗿𝘆) Task: abstract the scattered UI components, style tokens, and repeated interactions in this project into one unified Design System, a reusable and maintainable component library. Replace the hand-rolled one-off components with library components, route all new UI through the library with no bypassing, and add a review skill at the end. This is a refactor, so start in a worktree. For this task, write yourself a new end-to-end /goal: complete the whole plan, not just the next step, until the architecture, implementation, tests, review, and final result meet the standard. Split that goal into independent pieces, spawn as many parallel agents as needed to do it better and faster, and give each agent its own dedicated /goal that includes its expected deliverable, verification, and completion standard. Dispatch them concurrently, keep tracking progress in the right place, synthesize results as they return, resolve conflicts, continue implementation, run real-time validation after important steps, and finish with review, submission/commit when appropriate, and a final summary. Validation should cover the real end-to-end path, including browser/computer use, clicks, keyboard actions, and any necessary operation. Do not stop after partial progress unless blocked by missing credentials, destructive ambiguity, or conflicting requirements.
2. 𝗔𝗰𝗰𝗲𝘀𝘀𝗶𝗯𝗶𝗹𝗶𝘁𝘆 + 𝗺𝘂𝗹𝘁𝗶-𝗱𝗲𝘃𝗶𝗰𝗲 Task: make sure regular, low-vision, keyboard-only, and different-device users can all reliably complete the core actions. Start the app locally from the current commit, walk the core paths at desktop / tablet / phone viewports, and fix responsive issues (overlap / overflow / truncation / horizontal scroll), contrast, font scaling, keyboard (Tab / Enter / Space / Esc), focus states, and the a11y semantics of images / icon buttons / forms / status. Record the issues, the fixes, and the remaining risks, and add a review skill at the end. For this task, write yourself a new end-to-end /goal: complete the whole plan, not just the next step, until the architecture, implementation, tests, review, and final result meet the standard. Split that goal into independent pieces, spawn as many parallel agents as needed to do it better and faster, and give each agent its own dedicated /goal that includes its expected deliverable, verification, and completion standard. Dispatch them concurrently, keep tracking progress in the right place, synthesize results as they return, resolve conflicts, continue implementation, run real-time validation after important steps, and finish with review, submission/commit when appropriate, and a final summary. Validation should cover the real end-to-end path, including browser/computer use, clicks, keyboard actions, and any necessary operation. Do not stop after partial progress unless blocked by missing credentials, destructive ambiguity, or conflicting requirements.
3. 𝗚𝗶𝘃𝗲 𝗶𝘁 𝗮 DESIGN.md 𝗳𝗶𝗿𝘀𝘁 DESIGN.md is a concept from Google Stitch: a plain-text design system that spells out color, type scale, spacing, radius, shadow, and component rules, so the agent reads it before every UI build and the style stops drifting. AGENTS.md is how to build it, DESIGN.md is how it should look. Don't want to write one? Steal it. Someone scraped 73 real ones (Stripe / Linear / Airbnb / Vercel) into [github.com/VoltAgent/awesome-design-md](https://github.com/VoltAgent/awesome-design-md). Drop one in your root and tell the agent to make all UI follow DESIGN.md.
4. 𝗨𝗫 𝗮𝘂𝗱𝗶𝘁 (𝗹𝗲𝘁 𝗶𝘁 𝘀𝗲𝗲 𝘁𝗵𝗲 𝘀𝘁𝗮𝘁𝗲 𝗳𝗶𝗿𝘀𝘁) Act as a senior UX designer auditing this app's core flows. Walk the real user path end to end. Find: weak information hierarchy, friction, missing feedback, error-prone steps, empty and error states that were never built. Rate each by severity and give a concrete fix.
5. 𝗥𝗲𝗯𝘂𝗶𝗹𝗱 𝗼𝗻𝗲 𝗽𝗮𝗴𝗲 𝘄𝗶𝘁𝗵 𝗿𝗲𝗮𝗹 𝘁𝗮𝘀𝘁𝗲 Act as a product designer and rebuild this page to feel like Linear / Stripe. Pin down the style first: a restrained palette, a clear type scale, enough whitespace, consistent radius and shadow. Then build. Only touch the visuals, no new features, and walk me through every change.
6. 𝗚𝗲𝘁 𝗲𝘃𝗲𝗿𝘆 𝗶𝗻𝘁𝗲𝗿𝗮𝗰𝘁𝗶𝗼𝗻 𝗮𝗻𝗱 𝘀𝘁𝗮𝘁𝗲 𝗿𝗶𝗴𝗵𝘁 Act as a senior frontend engineer and get every interaction and state right. Cover: loading skeletons, empty states, error states, hover / focus / disabled, plus transitions and feedback on the key actions. Make it reusable and accessible, then deliver the component structure and full implementation.
7. 𝗕𝘂𝗶𝗹𝗱 𝗮 𝗹𝗮𝗻𝗱𝗶𝗻𝗴 𝗽𝗮𝗴𝗲 𝘁𝗵𝗮𝘁 𝗰𝗼𝗻𝘃𝗲𝗿𝘁𝘀 Act as a conversion-minded designer and build a landing page. Nail the one-line value prop and the single action you want from the visitor first. Then lay out: hero, social proof, feature sections, FAQ, CTA. Clean visuals, clear hierarchy, mobile-first. Deliver a complete, shippable page.
8. 𝗠𝗮𝗸𝗲 𝘁𝗵𝗲 𝗳𝗼𝗿𝗺 𝘂𝘀𝗮𝗯𝗹𝗲 Act as a senior frontend engineer and make this form usable. Cover: real-time validation, clear inline errors, sensible defaults, keyboard and autofill friendly, submitting and success / failure states, long forms split into steps. Deliver the full implementation.

</details>
