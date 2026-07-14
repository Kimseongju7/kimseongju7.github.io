---
title: "루프 시작하기 — Claude Code 팀이 정의하는 4가지 루프"
date: 2026-07-14 14:10:45 +0900
categories: [학습, 아티클]
tags: [claude-code, loops, agents, automation, workflow, skills]
slug: getting-started-with-loops
publish: true
---
요즘 코딩 에이전트에 프롬프트를 쓰는 대신 "루프를 설계하라"는 이야기가 많다. Claude Code 팀은 **루프를 "정지 조건이 충족될 때까지 작업 사이클을 반복하는 에이전트"** 로 정의하고, 트리거 방식·정지 방식·사용하는 Claude Code 프리미티브·적합한 작업 유형에 따라 네 가지로 분류한다. 모든 작업에 복잡한 루프가 필요한 건 아니므로, 가장 단순한 해법부터 시작해 이 패턴들을 선택적으로 쓰라는 것이 핵심이다.

## 턴 기반 루프 (Turn-based loops)

- **트리거**: 사용자의 프롬프트
- **정지 기준**: Claude가 작업을 완료했다고 판단하거나 추가 컨텍스트가 필요하다고 판단할 때
- **적합한 작업**: 정기적인 프로세스/일정에 속하지 않는 짧은 작업
- **사용량 관리**: 구체적인 프롬프트 작성, 스킬로 검증을 개선해 턴 수 줄이기

보내는 모든 프롬프트는 사용자가 각 턴을 지휘하는 수동 루프를 시작한다. Claude는 컨텍스트를 모으고, 행동하고, 작업을 점검하고, 필요하면 반복한 뒤 응답한다 — 이를 **에이전틱 루프(agentic loop)** 라 부른다. 예를 들어 좋아요 버튼을 만들어 달라고 하면 코드를 읽고, 수정하고, 테스트를 돌린 뒤 결과를 넘겨주고, 사용자가 수동으로 확인해 다음 프롬프트를 쓴다.

수동 점검 절차를 `SKILL.md`로 인코딩하면 Claude가 스스로 엔드투엔드 검증을 더 많이 할 수 있다. 결과를 보고·측정·상호작용할 수 있는 도구나 커넥터를 포함시키고, 점검이 정량적일수록 자가 검증이 쉬워진다. 예시 스킬(`verify-frontend-change`): 편집 성공만으로 UI 변경을 완료로 보고하지 말고, ① dev 서버를 띄워 브라우저에서 페이지 열기 ② 새 컨트롤을 직접 클릭해 상태 변화 확인 + 전후 스크린샷 ③ 브라우저 콘솔에 새 에러/경고 0건 확인 ④ Chrome Devtools MCP로 성능 트레이스 및 Core Web Vitals 감사 — 실패 시 고치고 1단계부터 재실행.

## 목표 기반 루프 (/goal)

- **트리거**: 실시간 수동 프롬프트
- **정지 기준**: 목표 달성 또는 최대 턴 수 도달
- **적합한 작업**: 검증 가능한 종료 기준이 있는 작업
- **사용량 관리**: 구체적인 완료 기준과 명시적 턴 상한("5번 시도 후 중단") 설정

복잡한 작업은 한 턴으로 부족하고, 에이전트는 반복(iterate)할 수 있을 때 더 잘한다. `/goal`로 "완료"의 정의를 내려 주면, Claude가 "이 정도면 됐다"고 스스로 판단해 일찍 멈추는 일을 막는다. Claude가 멈추려 할 때마다 평가자 모델(evaluator)이 조건을 확인하고, 목표 달성이나 지정 턴 수 도달 전까지 다시 작업으로 돌려보낸다. 테스트 통과 개수나 점수 임계치처럼 **결정론적 기준**이 특히 효과적이다.

```bash
/goal get the homepage Lighthouse score to 90 or above, stop after 5 tries.
```

## 시간 기반 루프 (/loop, /schedule)

- **트리거**: 지정한 시간 간격
- **정지 기준**: 사용자가 취소하거나 작업이 완료될 때(PR 머지, 큐 비움 등)
- **적합한 작업**: 반복 작업, 외부 환경/시스템과의 연동
- **사용량 관리**: 간격을 길게 잡거나 시간 대신 이벤트 기반으로 반응

매일 아침 Slack 메시지 요약처럼 입력만 바뀌는 반복 작업, 또는 코드 리뷰나 CI 실패를 받는 PR처럼 외부 시스템에 의존하는 작업에는 `/loop`로 프롬프트를 주기적으로 재실행할 수 있다.

```bash
/loop 5m check my PR, address review comments, and fix failing CI
```

`/loop`는 내 컴퓨터에서 실행되므로 컴퓨터를 끄면 멈춘다. `/schedule`로 루틴(routine)을 만들면 루프를 클라우드로 옮길 수 있다.

## 프로액티브 루프 (Proactive loops)

- **트리거**: 이벤트나 스케줄 — 실시간 개입하는 사람 없음
- **정지 기준**: 각 태스크는 목표 달성 시 종료, 루틴 자체는 사용자가 끌 때까지 실행
- **적합한 작업**: 버그 리포트, 이슈 트리아지, 마이그레이션, 의존성 업그레이드 같은 잘 정의된 반복 작업 스트림
- **사용량 관리**: 루틴은 더 작고 빠른 모델로, 판단이 필요한 부분만 가장 유능한 모델로 라우팅

위 프리미티브들을 **auto mode**, **dynamic workflows**(research preview) 같은 기능과 조합해 장기 실행 루프를 구성할 수 있다. 예를 들어 피드백 처리에는 ① `/schedule`(research preview)로 새 리포트를 확인하는 루틴 실행 ② `/goal`로 완료 조건 정의 + 스킬로 검증 방법 문서화 ③ dynamic workflows로 리포트 트리아지·수정·리뷰 에이전트 오케스트레이션 ④ auto mode로 권한 질문 없이 루틴 실행.

```bash
/schedule every hour: check the project-feedback channel for bug reports. /goal: don't stop until every report found this run is triaged, actioned, and responded to. When fixing a bug, use a workflow to explore three solutions in parallel worktrees and have a judge adversarially review them.
```

## 코드 품질 유지하기

루프 출력의 품질은 그것을 둘러싼 시스템에 달려 있다.

- **코드베이스 자체를 깨끗하게**: Claude는 코드베이스에 이미 존재하는 패턴과 컨벤션을 따른다.
- **스스로 검증할 수단 제공**: 팀 기준에서 "좋은 결과"가 무엇인지 [스킬](https://code.claude.com/docs/en/skills)로 인코딩한다.
- **문서를 쉽게 접근 가능하게**: 프레임워크·라이브러리 문서에 최신 베스트 프랙티스가 있다.
- **코드 리뷰는 두 번째 에이전트로**: 새 컨텍스트의 리뷰어는 메인 에이전트의 추론에 영향받지 않아 덜 편향된다. 내장 `/code-review` 스킬이나 GitHub용 [Code Review](https://code.claude.com/docs/en/code-review)를 쓴다.

개별 결과가 기준에 못 미치면 그 건만 고치지 말고, 이후 모든 반복이 나아지도록 시스템에 인코딩하라.

## 토큰 사용량 관리

- **작업에 맞는 프리미티브와 모델 선택**: 작은 작업엔 다중 에이전트/루프가 필요 없고, 더 싸고 빠른 모델로 충분한 작업도 있다.
- **명확한 성공·정지 기준 정의**: "완료"를 구체적으로 정의할수록 Claude가 (너무 이르지 않게) 더 빨리 답에 도달한다.
- **대규모 실행 전 파일럿**: dynamic workflows는 수백 개 에이전트를 띄울 수 있으니, 작은 조각으로 먼저 사용량을 가늠하라.
- **결정론적 작업엔 스크립트**: 스크립트 실행이 단계별 추론보다 싸다. 예: PDF 스킬에 폼 채우기 스크립트를 포함해 매번 코드를 재도출하지 않기.
- **루틴을 필요 이상 자주 돌리지 않기**: 관찰 대상이 바뀌는 빈도에 간격을 맞춘다.
- **사용량 검토**: `/usage`는 스킬·서브에이전트·MCP별 최근 사용량을, 인자 없는 `/goal`은 지금까지의 턴 수·토큰 사용량을, `/workflows`는 에이전트별 토큰 사용량을 보여 주며 언제든 에이전트를 중단할 수 있다.

## 시작하기

| 루프 | 넘기는 것 | 쓰는 시점 | 도구 |
| --- | --- | --- | --- |
| 턴 기반 | 점검(check) | 탐색하거나 결정하는 중일 때 | 커스텀 검증 스킬 |
| 목표 기반 | 정지 조건 | "완료"가 어떤 모습인지 알 때 | /goal |
| 시간 기반 | 트리거 | 작업이 프로젝트 밖에서 일정에 따라 발생할 때 | /loop, /schedule |
| 프로액티브 | 프롬프트 | 작업이 반복적이고 잘 정의돼 있을 때 | 위 전부 + dynamic workflows |

이미 하고 있는 일에서 내가 병목인 작업 하나를 골라, 어떤 조각을 넘길 수 있을지 물어보라: 검증 체크를 글로 쓸 수 있는가? 목표가 충분히 명확한가? 작업이 일정에 따라 도착하는가? 아이디어가 생기면 루프를 돌리고, 어디서 멈추거나 과하게 나가는지 관찰하며 주저 없이 반복 개선하라. 자세한 내용은 Claude Code 문서의 agents, loop, schedule(routines), goal, dynamic workflows 페이지 참고. 이 글은 [@delba_oliveira](https://x.com/@delba_oliveira)가 작성했다.

## 관련 노트

- [[what-is-a-loop-anyway]] — "루프"라는 한 단어에 숨은 네 가지 아키텍처를 지도로 정리한 이론편
- [[art-of-loop-engineering]] — LangChain이 정리한 4단계 루프 스택(에이전트·검증·이벤트·힐 클라이밍)
- [[Loop Engineering|루프 엔지니어링]] — 루프를 설계·중첩하는 개념 용어

> 원문: [Getting started with loops](https://x.com/ClaudeDevs/status/2074208949205881033)
> 원본 클립: [[2026-07-14-Getting started with loops]]
