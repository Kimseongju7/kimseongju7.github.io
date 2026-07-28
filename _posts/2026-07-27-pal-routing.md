---
title: 'PAL 라우팅'
date: 2026-07-27 11:16:00 +0900
categories: [학습, 용어]
tags: [ouroboros, llm, 모델라우팅, 비용최적화, ai-agent]
description: 'Ouroboros의 ModelRouter는 3개 티어를 둔다.'
---
## 한 줄 정의
PAL = **Progressive Auto-escalation** — 싼 모델로 먼저 시도하고, 실패가 쌓이면 한 단계씩 비싼 모델로 자동 승급(성공이 쌓이면 다시 강등)하는 모델 라우팅 전략.

## 자세히
LLM 에이전트에서 비용의 대부분은 "쉬운 일까지 비싼 모델로 처리하는" 데서 나온다. 반대로 전부 싼 모델로 돌리면 어려운 작업에서 무한 재시도에 빠진다. PAL은 이 둘 사이를 **작업 난이도에 따라 동적으로** 오간다. 사람이 미리 "이 작업은 Opus, 저건 Haiku"라고 정해두지 않아도, 실제 실패/성공 이력이 등급을 결정한다.

[Ouroboros](/posts/ouroboros-usage/)의 `ModelRouter`는 3개 티어를 둔다.

| 티어 | 상대 비용(cost_factor) | 쓰임 |
|---|---|---|
| `frugal` | 1 | 기본값. 단순·기계적 작업 |
| `standard` | 10 | 중간 난이도 |
| `frontier` | 30 | 창의적 재구성, 어려운 판단 |

동작 규칙(기본값 기준):
- 시작 티어는 `default_tier: frugal`
- `escalation_threshold: 2` — **2번째 재시도부터** 승급 시작. 이후 재시도 1회당 한 단계씩 오르고, `frontier`가 천장이다
- `downgrade_success_streak: 5` — 현재 티어에서 5번 연속 성공하면 한 단계 **내려간다**. 어려운 구간이 지나면 다시 싸지는 구조
- 티어별로 `models`(provider + model id), `intelligence_range`, `use_cases` 를 설정으로 지정한다

단계별 오버라이드도 있다. 예를 들어 [측면 사고](/posts/lateral-thinking/) 호출은 기본이 `frontier`인데, 창의적 재구성에는 애초에 높은 모델 능력이 필요하다고 보기 때문이다. 반대로 인터뷰 명료화(clarification) 단계는 별도 티어를 지정할 수 있다.

핵심 아이디어는 **"똑똑한 모델을 쓰자"가 아니라 "언제 똑똑한 모델이 필요한지를 시스템이 판단하게 하자"**는 것이다. 난이도는 사람이 미리 알기 어렵지만, 실패 횟수는 실행 중에 정확히 관측된다.

## 예시
```
AC-3 "CSV 파서 구현" 실행

시도 1  → frugal    (실패: 인용부호 escape 처리 누락)
시도 2  → standard  (escalation_threshold=2 도달, 한 단계 승급 / 실패)
시도 3  → frontier  (성공)

이후 AC-4 ~ AC-8 을 frontier 로 5연속 성공
  → downgrade_success_streak=5 충족 → standard 로 강등
```

## 관련
- [Seed 스펙](/posts/seed-spec/) — PAL이 실행하는 대상(AC)이 정의된 곳
- [측면 사고](/posts/lateral-thinking/) — 기본 티어가 `frontier`로 지정된 대표 단계
- [우로보로스 사용법](/posts/ouroboros-usage/) — `--efficiency-mode`, `--frugality-assurance` 등 관련 플래그
- [Q00/ouroboros — 프롬프트를 멈추고 명세를 시작하는 Agent OS](/posts/ouroboros-agent-os/) — 런타임·라우팅 계층 설명
- [Loop Engineering](/posts/loop-engineering/) — 루프에서 "토큰 폭발"을 막는 문제의식과 직접 연결되는 개념
