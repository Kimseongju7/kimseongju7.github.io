---
title: '결정화'
date: 2026-07-27 11:13:00 +0900
categories: [학습, 용어]
tags: [ouroboros, spec-first, 요구사항, ai-agent, workflow]
description: '결정화는 그 중간 단계를 명시적인 산출물로 만든다. 액체(애매한 의도)를 그대로 실행에 흘려보내지 않고, 먼저 결정(고정된 스펙)으로 굳힌 다음 그 결정을 기준으로 실행한다. Ouroboros에서는 소크라테스 인터뷰가 숨은 가정…'
---
## 한 줄 정의
Crystallization — 머릿속의 애매한 의도를 질문으로 파헤쳐 **더 이상 해석의 여지가 없는 고정된 명세**로 굳히는 과정.

## 자세히
AI 코딩에서 실패는 대개 **출력**이 아니라 **입력**에서 난다. "로그인 기능 만들어줘" 같은 요청은 사람 머릿속에선 완결된 그림이지만, 문장으로 내보내는 순간 정보 대부분이 떨어져 나간다. 소셜 로그인이 필요한지, 세션 만료는 몇 분인지, 실패 시 어떻게 되는지 — 이런 건 말한 적 없으니 모델이 매번 다르게 채운다. 같은 프롬프트를 두 번 넣어도 다른 결과가 나오는 이유가 이것이다.

결정화는 그 중간 단계를 명시적인 산출물로 만든다. 액체(애매한 의도)를 그대로 실행에 흘려보내지 않고, 먼저 결정(고정된 스펙)으로 굳힌 다음 그 결정을 기준으로 실행한다. [Ouroboros](/posts/ouroboros-usage/)에서는 [소크라테스 인터뷰](/posts/socratic-interview/)가 숨은 가정을 질문으로 끄집어내고, 그 답을 `seed-architect` 에이전트가 불변 YAML인 [Seed 스펙](/posts/seed-spec/)으로 굳히는 단계를 가리킨다.

결정화가 주는 실질적 이득은 **기준점**이다. 스펙이 고정돼야 "지금 만들어진 게 원래 요구사항과 얼마나 어긋났나"를 측정할 수 있다(드리프트 감지). 즉흥 프롬프트만으로 굴리면 어긋난 건지 원래 그런 건지 판단할 근거 자체가 없다. 검증(evaluate)도, 되감기(rewind)도, 재실행도 모두 고정된 스펙이 있어야 성립한다.

주의할 점은 결정화가 "문서를 많이 쓰는 것"과 다르다는 것이다. 핵심은 분량이 아니라 **가정을 드러내 합의로 바꾸는 것**이다. 결정화되지 않은 요구사항은 문서가 길어도 여전히 액체다.

## 예시
```
결정화 전:  "할 일 관리 CLI 만들어줘"

인터뷰:     저장은 로컬 파일? DB?     →  로컬 JSON
            동시 사용자 있나?          →  단일 사용자
            완료 항목은 삭제? 보관?    →  보관 (--archived 로 조회)

결정화 후 (Seed):
  acceptance_criteria:
    - id: AC-1  "todo add <text> 로 항목 추가, ~/.todo.json 에 append"
    - id: AC-2  "todo done <id> 로 archived=true 표시, 목록에서 숨김"
    - id: AC-3  "todo list --archived 로 보관 항목 조회"
  constraints:
    - "외부 DB 의존성 없음"
```

## 관련
- [Seed 스펙](/posts/seed-spec/) — 결정화의 결과물인 불변 YAML 명세
- [소크라테스 인터뷰](/posts/socratic-interview/) — 결정화를 수행하는 질문 단계
- [우로보로스 사용법](/posts/ouroboros-usage/) — 결정화 엔진을 실제로 굴리는 방법
- [Q00/ouroboros — 프롬프트를 멈추고 명세를 시작하는 Agent OS](/posts/ouroboros-agent-os/) — "인터뷰 → 결정화 → 실행 → 평가 → 진화" 전체 워크플로
- [결정 기록](/posts/decision-record/) — 왜 그렇게 정했는지를 남긴다는 점에서 결정화와 짝을 이루는 습관
- [MVP (Minimum Viable Product)](/posts/mvp/) — 무엇을 만들지 좁혀 확정한다는 점에서 통하는 개념
- [명세 주도 개발(SDD)](/posts/spec-driven-development/) — 결정화를 개발 프로세스의 1급 단계로 승격시킨 방법론
