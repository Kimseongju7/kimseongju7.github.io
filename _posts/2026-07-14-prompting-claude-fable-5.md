---
title: 'Claude Fable 5 프롬프팅 가이드'
date: 2026-07-14 14:15:40 +0900
categories: [개발, AI]
tags: [claude, prompt-engineering, fable-5, anthropic, agents, llm]
description: 'Claude Fable 5(및 Claude Mythos 5)에 특화된 프롬프팅·스캐폴딩 패턴을 다루는 Anthropic 공식 가이드로, 장기 자율성·강한 지시 이행·병렬 서브에이전트 등 이전 모델과 달라진 지점을 정리한다.'
---
Claude Fable 5(및 Claude Mythos 5)에 특화된 프롬프팅·스캐폴딩 패턴을 다루는 Anthropic 공식 가이드다. Fable 5는 이전 모델에게는 너무 복잡하거나 장기적이거나 모호했던 문제, 특히 사람이 몇 시간~몇 주 걸리는 end-to-end 작업에 강하며, 가장 좋은 성과를 내는 팀들은 가장 어려운 미해결 문제에 이 모델을 투입한다.

## 능력 개선 (Opus 4.8 대비)

- **장기 자율성(long-horizon autonomy)**: 며칠에 걸친 목표 지향적 실행을 유지하고 긴 작업에서도 지시를 잘 기억한다.
- **잘 명세된 복잡한 문제의 첫 시도 정확도**: 초기 테스터들이 며칠 걸리던 시스템을 단일 패스로 구현했다고 보고.
- **비전**: 밀도 높은 기술 이미지·웹 앱·스크린샷을 훨씬 정확하게(종종 더 적은 출력 토큰으로) 해석하며, 뒤집히거나 흐릿한 이미지를 bash·크롭 도구로 처리하도록 학습됨.
- **엔터프라이즈 워크플로**: 재무 분석, 스프레드시트, 슬라이드, 문서에서 지시를 따르고 범위를 지키며 전문가급 산출물 생성.
- **코드 리뷰·디버깅**: 버그 발견 재현율이 눈에 띄게 높음(코드베이스·레포 히스토리 검색 포함).
- **모호성 탐색**: 복잡하고 여러 갈래인 요청에서 다음 단계를 스스로 판단.
- **위임과 협업**: 병렬 서브에이전트 디스패치와 장기 실행 서브에이전트·피어 에이전트와의 지속적 소통이 훨씬 안정적.

단, 공격적 사이버보안이나 생물학·생명과학 작업은 대상이 아니며 해당 도메인 요청은 `stop_reason: "refusal"`을 반환할 수 있다. 안전 분류기가 공격 도구 제작, 생물학 실험 방법, 요약된 사고(thinking) 추출을 타겟팅하며, 무해한 보안·생명과학 작업도 걸릴 수 있으므로 거절된 요청은 Opus 4.8로의 서버/클라이언트 폴백을 구성해 자동 재라우팅한다.

## 기본적으로 더 긴 턴

높은 effort 설정에서 어려운 작업의 단일 요청이 수 분씩 돌 수 있고, 자율 실행은 몇 시간까지 이어진다. 마이그레이션 전에 클라이언트 타임아웃·스트리밍·진행 표시 UI를 조정하고, 블로킹 대신 스케줄 잡 등으로 비동기 확인하도록 하네스 재구성을 고려하라. 모호한 작업에서 과잉 계획을 막는 지시 예:

```
When you have enough information to act, act. Do not re-derive facts already established in the conversation, re-litigate a decision the user has already made, or narrate options you will not pursue in user-facing messages. If you are weighing a choice, give a recommendation, not an exhaustive survey. This does not apply to thinking blocks.
```

## 모든 effort 레벨을 고려하라

effort는 Fable 5에서 지능·지연시간·비용 트레이드오프의 주 컨트롤이다. 대부분 작업은 `high`를 기본으로, 가장 능력에 민감한 워크로드는 `xhigh`, 루틴 작업은 `medium`/`low`를 쓴다. 낮은 effort도 이전 모델의 `xhigh`를 넘는 경우가 많다. 높은 effort에서 요청하지 않은 정리·리팩터링을 막으려면:

```
Don't add features, refactor, or introduce abstractions beyond what the task requires. ... Don't add error handling, fallbacks, or validation for scenarios that cannot happen. Trust internal code and framework guarantees. Only validate at system boundaries (user input, external APIs). Don't use feature flags or backwards-compatibility shims when you can just change the code.
```

## 강한 지시 이행

지시 이행이 좋아져서 행동 하나하나를 열거하는 대신 짧은 지시로 조종할 수 있다. 예를 들어 과잉 설명(추구하지 않을 옵션 나열, 장황한 근본 원인 설명, 과하게 구조화된 PR 설명 등)에는 짧은 간결성 지시가 목록 나열만큼 효과적이다:

```
Lead with the outcome. Your first sentence after finishing should answer "what happened" or "what did you find" ... The way to keep output short is to be selective about what you include, not to compress the writing into fragments, abbreviations, arrow chains like A → B → fails, or jargon.
```

체크포인트 행동도 마찬가지로, 정말 사용자가 필요한 경우만 멈추게 하려면:

```
Pause for the user only when the work genuinely requires them: a destructive or irreversible action, a real scope change, or input that only they can provide. If you hit one of these, ask and end the turn, rather than ending on a promise.
```

## 장기 실행 시 진행 보고를 근거에 묶기

자율 실행 중에는 진행 주장을 실제 툴 결과와 대조하도록 지시하라. Anthropic 테스트에서 이 지시가 조작된 상태 보고를 유도하도록 설계된 태스크에서도 허위 보고를 거의 없앴다:

```
Before reporting progress, audit each claim against a tool result from this session. Only report work you can point to evidence for; if something is not yet verified, say so explicitly. Report outcomes faithfully: if tests fail, say so with the output; if a step was skipped, say that ...
```

## 경계를 명시하라

Fable 5는 가끔 요청하지 않은 행동(부탁하지 않은 이메일 초안, 방어적 git 브랜치 백업 생성)을 할 수 있다. 해야 할 것과 하지 말아야 할 것을 명시적으로 정의하라:

```
When the user is describing a problem, asking a question, or thinking out loud rather than requesting a change, the deliverable is your assessment. Report your findings and stop. Don't apply a fix until they ask for one. ...
```

## 병렬 서브에이전트

Fable 5는 이전 모델보다 병렬 서브에이전트를 훨씬 적극적으로 쓴다. 서브에이전트를 자주 쓰고, 위임이 적절한 시점을 명시하고, 각 서브에이전트가 끝날 때까지 블로킹하기보다 오케스트레이터-서브에이전트 간 비동기 소통을 선호하라. 컨텍스트를 유지하는 장수(long-lived) 서브에이전트는 캐시 읽기로 시간·비용을 아끼고 가장 느린 서브에이전트에 병목이 걸리는 것을 피한다.

## 메모리 시스템 구축

Fable 5는 이전 실행의 교훈을 기록하고 참조할 수 있을 때 특히 잘한다. 마크다운 파일 하나로도 충분하다:

```
Store one lesson per file with a one-line summary at the top. Record corrections and confirmed approaches alike, including why they mattered. Don't save what the repo or chat history already records; update an existing note rather than creating a duplicate; delete notes that turn out to be wrong.
```

기존 히스토리에서 부트스트랩하려면 과거 세션을 리뷰시키면 된다("서브에이전트로 핵심 테마와 교훈을 뽑아 [X]에 저장하라").

## 드문 조기 중단 사례

긴 세션 깊숙이에서 "이제 X를 실행하겠다"는 텍스트만 남기고 툴 호출 없이 턴을 끝내거나, 진행해도 되는데 허락을 구할 때가 가끔 있다. "continue"나 "끝까지 진행해" 한 마디면 충분하다. 자율 파이프라인에는 시스템 리마인더를 추가하라:

```
You are operating autonomously. The user is not watching in real time and cannot answer questions mid-task ... Before ending your turn, check your last paragraph. If it is a plan, an analysis, a question, a list of next steps, or a promise about work you have not done, do that work now with tool calls. ...
```

## 드문 컨텍스트 예산 걱정 사례

아주 긴 세션에서 새 세션을 제안하거나 요약·핸드오프를 권하거나 스스로 작업을 줄일 수 있는데, 대부분 하네스가 남은 토큰 카운트다운을 모델에게 보여줄 때 발생한다. 가능하면 명시적 컨텍스트 예산 수치를 노출하지 말고, 노출해야 한다면 "컨텍스트는 충분하니 중단·요약·새 세션 제안을 하지 말고 계속하라"는 안심 문구를 넣는다.

## 요청만이 아니라 이유를 줘라

Fable 5는 요청의 의도를 이해할 때 더 잘한다. 특히 여러 워크스트림을 다루는 장기 실행 에이전트에는 왜 요청하는지 맥락을 제공하라: "나는 [누구를 위한] [더 큰 작업]을 하고 있다. 그들에게는 [산출물이 가능케 하는 것]이 필요하다. 그걸 염두에 두고: [요청]."

## 사용자와 소통할 때의 가독성

긴 에이전틱 대화에서 화살표 체인 축약, 깊은 구현 디테일, 사용자가 못 본 사고 참조, 과한 기술 용어 등 따라가기 힘든 텍스트가 나올 수 있다. 소통 스타일 부록으로 완화한다. 핵심: 툴 호출 사이의 축약은 괜찮지만 최종 요약은 그 과정을 못 본 독자를 위한 것이므로, 결과부터 완전한 문장으로 쓰고, 작업 중 만든 어휘·축약·화살표 체인은 버리고, 짧음과 명확함 중에서는 명확함을 골라야 한다.

## send-to-user 툴 만들기

길고 비동기적인 에이전트에는 턴을 끝내지 않고 사용자에게 원문 그대로 메시지를 전달하는 클라이언트 사이드 툴을 줘라. 산출물(코드 스니펫, 메시지 초안), 구체적 수치의 진행 업데이트, 루프 중 질문에 대한 직접 답변 등에 쓴다. 툴 입력은 절대 요약되지 않으므로 내용이 그대로 도착한다.

```
{
  "name": "send_to_user",
  "description": "Display a message directly to the user. Use this for progress updates, partial results, or content the user must see exactly as written before the task finishes.",
  "input_schema": {
    "type": "object",
    "properties": {
      "message": { "type": "string", "description": "The content to display to the user." }
    },
    "required": ["message"]
  }
}
```

툴 정의만으로는 부족하고, 시스템 프롬프트에 호출을 유도하는 지시를 함께 넣어야 한다. 내레이션이나 내부 추론을 이 툴로 보내지 말 것 — 과용하면 목적이 무색해진다.

## 마이그레이션 팁

- **난이도 상한에서 시작하라.** 이전 모델에 맡기던 것보다 어려운 작업을 골라 스코핑·질문·실행을 시켜라.
- **장기 실행 프롬프트에 자기 검증을 명시하라.** 컨텍스트가 신선한 별도의 검증자 서브에이전트가 자기 비평보다 낫다.
- **기존 프롬프트와 스킬을 리팩터링하라.** 이전 모델용 스킬은 Fable 5에게 과하게 지시적(prescriptive)이어서 품질을 떨어뜨릴 수 있다. 기본 성능이 더 좋다면 오래된 지시를 제거하라. Fable 5는 작업 중 배운 것을 바탕으로 스킬을 즉석에서 잘 업데이트하기도 한다.
- **추론을 응답에 재현하라고 지시하지 마라.** 내부 추론을 응답 텍스트로 옮겨 쓰게 하는 지시는 `reasoning_extraction` 거절 카테고리를 트리거해 Opus 4.8 폴백을 늘릴 수 있다. 추론 가시성이 필요하면 adaptive thinking의 구조화된 `thinking` 블록을 읽고, 진행 상황은 send-to-user 툴로 표면화하라.

## 관련 노트

- [claude-code-model-and-effort](/posts/claude-code-model-and-effort/) — Claude Code의 모델·effort 선택 전략
- [claude-code-artifacts](/posts/claude-code-artifacts/) — 세션 작업을 시각 아티팩트로 표면화하는 기능
- [claude-code-multi-agent-sessions](/posts/claude-code-multi-agent-sessions/) — 병렬 서브에이전트 오케스트레이션
- [headroom-context-compression](/posts/headroom-context-compression/) — 컨텍스트 예산을 절감하는 압축 레이어

> 원문: [Prompting Claude Fable 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5)
> 원본 클립: 2026-07-14-Prompting Claude Fable 5
