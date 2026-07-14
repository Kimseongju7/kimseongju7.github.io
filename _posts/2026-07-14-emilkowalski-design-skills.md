---
title: 'emilkowalski/skills — 디자인 엔지니어를 위한 스킬 모음'
date: 2026-07-14 14:05:54 +0900
categories: [개발, 도구]
tags: [design, animation, skills, ai-agent, ui, frontend]
description: 'Vercel·Linear 출신 디자인 엔지니어 Emil Kowalski가 만든, 에이전트의 애니메이션·디자인 실수를 고쳐주는 AI 스킬 모음 정리.'
---
Vercel·Linear 출신 디자인 엔지니어 Emil Kowalski가 만든 AI 에이전트용 스킬 모음. 애니메이션과 디자인에서 "올바른 선택"에 더 빨리 도달하도록 돕는 것이 목적이며, 에이전트가 흔히 저지르는 작은 실수들을 나열하고 고치는 법을 알려준다.

## 왜 쓰는가 — 에이전트는 취향(taste)이 없다

- 에이전트는 애니메이션 재료를 잘못 고르는 일이 많다. 예: 등장(enter) 애니메이션에 `ease-out`을 써야 하는데 `ease-in`을 쓰거나, 반투명 그림자 대신 불투명 실선 보더를 고르는 식.
- 이런 작은 것들이 쌓여 인터페이스를 훌륭하게도, 그저 그렇게도 만든다.
- 저자의 글 "Agents with Taste"에서 설명하듯, 이 스킬들은 에이전트가 저지를 수 있는 실수 목록과 수정법을 담고 있다 — "슬롭(slop)의 바다에서 돋보이는 지름길".
- 저자의 관점: 이 스킬들은 도메인 전문성의 부산물이다. AI는 전문성을 대체하지 않고 증폭시키므로, 코딩·디자인 등 어느 분야든 전문성을 기르는 것이 여전히 매우 가치 있다.

## 설치

```
npx skills@latest add emilkowalski/skills
```

## 스킬 목록

- **emil-design-eng** — 메인 스킬. 대부분 애니메이션이고 일부 디자인 조언 포함.
- **review-animations** — 저자의 규칙에 따라 애니메이션을 엄격하게 리뷰.
- **improve-animations** — 코드베이스 전체의 애니메이션을 감사(audit)하고, 어떤 에이전트든 실행할 수 있는 우선순위화된 자기완결적 계획을 생성.
- **animation-vocabulary** — 올바른 용어로 원하는 것을 정확히 전달해 AI에서 더 나은 애니메이션을 끌어내는 스킬.
- **apple-design** — Apple WWDC 디자인 토크에서 추출한 인터페이스 디자인·유려한 모션 원칙을 웹용으로 번역한 것.

## improve-animations 동작 방식

shadcn/improve에서 영감을 받은 구조: 가장 유능한 모델로 감사를 하고, 실행은 더 저렴한 모델에 넘긴다.

- 단일 diff가 아니라 코드베이스 전체를 조사한다.
- 8개 카테고리로 감사: 목적·빈도, 이징·지속시간, 물리성(physicality), 중단 가능성(interruptibility), 성능, 접근성, 응집성, 놓친 기회.
- 우선순위화된 발견 사항 표를 제시하고, 선택한 항목에 대해 `plans/` 폴더에 자기완결적 계획(정확한 파일, 커브, 지속시간 + 느낌 점검)을 작성한다. 다른 에이전트가 컨텍스트나 취향 없이도 실행할 수 있다.
- 소스 코드는 직접 건드리지 않는다.

```
> improve the animations in this codebase
> improve-animations quick        # 핫스팟만
> improve-animations performance  # 카테고리 하나만
> improve-animations plan add press feedback to all buttons
> improve-animations execute plans/001-fix-dropdown-easing.md
```

## 관련 노트

- [interface-design-claude-code](/posts/interface-design-claude-code/) — 같은 계열의 UI 디자인 엔지니어링 스킬
- [디자인 시스템 프롬프트 모음](/posts/design-system-prompts/) — 디자인 프롬프트 참고
- [alirezarezvani-claude-skills](/posts/alirezarezvani-claude-skills/) — 대규모 스킬 라이브러리(디자인 스킬 포함)
- [mattpocock-skills](/posts/mattpocock-skills/) — 코딩 에이전트용 스킬 모음

> 원문: [emilkowalski/skills: Skills for Design Engineers.](https://github.com/emilkowalski/skills)
> 원본 클립: 2026-07-14-emilkowalskiskills Skills for Design Engineers.
