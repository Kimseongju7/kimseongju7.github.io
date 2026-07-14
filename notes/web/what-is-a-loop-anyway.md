---
title: "도대체 루프가 뭔데? — 한 단어 뒤에 숨은 네 가지 아키텍처"
date: 2026-07-14 14:11:54 +0900
categories: [학습, 아티클]
tags: [loops, agents, ai, software-factory, autoresearch, loop-engineering]
slug: what-is-a-loop-anyway
publish: true
---
AI 엔지니어링 세계가 이번 달 새로 사랑에 빠진 단어 "루프(the loop)"는 적어도 네 가지 서로 다른 것을 의미한다. 루프를 말하는 사람들은 같은 것을 이야기하고 있지 않다 — 이 글은 그 한 단어 뒤에 숨은 네 가지 아키텍처를 지도로 그려 본 시도다.

## 배경: 하이프 사이클의 정점

- 6월 7일 Peter Steinberger는 코딩 에이전트에 프롬프트를 쓰지 말고 에이전트에 프롬프트를 넣는 루프를 설계해야 한다고 포스팅했다.
- 같은 주 Anthropic의 Boris Cherny는 무대에서 "나는 이제 Claude에 프롬프트하지 않는다. 루프를 쓰고, 루프가 일한다"고 말했다.
- Addy Osmani가 6월 7일 [Loop Engineering](https://addyosmani.com/blog/loop-engineering/)을, swyx가 6월 12일 [Loopcraft: The Art of Stacking Loops](https://www.latent.space/p/ainews-loopcraft-the-art-of-stacking)를, LangChain이 6월 16일 [The Art of Loop Engineering](https://www.langchain.com/blog/the-art-of-loop-engineering)을 발표했다.
- 이어진 AI Engineer World's Fair(AIEWF)에서 이 단어가 메인 스테이지를 지배했다. swyx의 키노트는 Loopcraft였고, 소프트웨어 팩토리 전용 트랙이 있었으며, 7월 2일 폐막은 루프 하이프가 실제로 작동하는 것을 앞질렀는지에 대한 1시간 토론이었다.

## 1. 실행 루프(execution loop): 에이전트 자신의 행동-관찰 사이클

"에이전트"라 할 때 대부분이 떠올리는 루프: 도구를 호출하고, 결과를 읽고, 다음 행동을 정하고, 더 이상 호출할 도구가 없을 때까지 반복. Addy가 말하는 내부 실행 루프이고, 엔지니어링할 수 있는 가장 안쪽 루프다(swyx의 스택에는 토큰 루프도 있지만 그건 설계 대상이 아니라 모델의 일부다).

- **반복 대상**: 하나의 태스크 안의 단계들
- **종료 신호**: 환경 피드백 — 테스트 출력, API 응답, 파일 내용
- 사람은 루프 중간에는 대개 없고 경계(계획 승인, 결과 리뷰)에 나타난다. 또한 실제 완료 여부와 무관하게 에이전트가 끝났다고 판단하면 끝난다 — 이 문제의 첫 해법이, 에이전트의 말을 믿지 않는 바깥 루프로 감싸는 것이었다.

## 2. 태스크 루프(task loop): 스펙이 충족될 때까지 에이전트 재시작

이름을 얻은 첫 루프, Geoffrey Huntley의 **Ralph Loop**. AIEWF 메인 스테이지에서 Keycard의 Allie Howe가 그의 글 [everything is a ralph loop](https://ghuntley.com/loop/)을 인용하며 소프트웨어 팩토리 트랙을 열었다. Ralph 루프는 같은 스펙에 대해 코딩 에이전트를 반복 재시작하되, 매 반복마다 완전히 새로운 컨텍스트 윈도를 할당하고 루프당 정확히 하나의 태스크만 수행한다. 겉보기 낭비가 핵심이다: 전체 스펙을 매번 다시 먹이면 장기 세션을 조용히 망가뜨리는 컨텍스트 부패(context rot)와 컴팩션 이벤트를 방지한다.

- **반복 대상**: 단일 산출물(artifact)
- **종료 신호**: 스펙 준수와 테스트 통과
- 사람은 스펙을 쓰고 완료를 판정한다. Geoffrey의 표현으로 사람에겐 하나의 일이 더 있다: 루프를 지켜보며 실패 패턴을 발견하고 재발하지 않게 고치는 것. 폐막 토론에서 그는 이 역할을 기차를 궤도 위에 유지하는 게 전부인 기관차 기관사에 비유했다.

## 3. 프로덕트 루프(product loop): 소프트웨어 팩토리

AIEWF에서 가장 목소리가 컸던 버전. Factory의 Tereza Tížková는 소프트웨어 팩토리를 "루프 전체, 자율성을 갖춘 소프트웨어 개발 라이프사이클 전체"로 정의했고, Warp의 Zach Lloyd는 그 라이프사이클을 구체화했다: 트리아지, 스펙, 구현, 리뷰, 검증, 배포, 모니터링. Zach의 주장은 소프트웨어 엔지니어링이 팩토리 엔지니어링이 되고, "제품을 만드는 것을 만드는" 일을 하게 된다는 것이다.

- Warp는 이를 도그푸딩 중: 자사 오픈소스 저장소를 팩토리 플랫폼 Oz의 통제 아래 두었고, 저위험 저장소부터 시작해 자동 PR 머지 비율을 20%에서 60%를 향해 올려 간다.
- Anthropic도 내부에서 같은 실험을 하는 것으로 보인다: [제품 팀 코드의 65%](https://www.anthropic.com/news/introducing-claude-tag)가 내부 버전 Claude Tag로 만들어진다고 밝혔고, Mike Krieger는 그 사용을 위임형·능동형으로 묘사했다 — "이 버그 고쳐"가 아니라 코드베이스의 이 부분을 책임지고, 이 피드백 채널을 모니터링하고, 스스로 태스크를 집어 가는 방식.
- **반복 대상**: 코드베이스와 백로그, 지속적으로
- **종료 신호**: 코드베이스 바깥에서 온다 — 새 이슈, 프로덕션 로그, 사용자 피드백, 리뷰 결과. 사람의 역할은 구성 가능(configurable)해진다: 자동화할 라이프사이클 부분과 사람이 개입할 지점을 고르며, 고위험 변경의 코드 리뷰를 사람이 유지할지 등은 조직마다 다르다.

## 4. 시스템 루프(system loop): 오토리서치

Introspection의 Roland Gavrilescu가 **autoresearch**라 부르는 것으로, [Latent Space 인터뷰](https://www.latent.space/p/autoresearch-introspection)의 프레이밍이 가장 깔끔하다: 내부 루프는 사용자 대면 작업을 하는 주 시스템이고, 외부 루프는 주 시스템을 연구·유지한다. 프롬프트, 하네스, 모델 선택, 그리고 평가(eval) 자체를 반복 개선한다. 그의 한 줄 요약: "루프가 곧 제품이다."

- 최소 사례: Andrej Karpathy의 2026년 3월 autoresearch — 약 630줄의 Python으로 GPU 한 장에서 하룻밤 새 50번의 가설-수정-평가 실험을 돌렸다.
- 출시 사례: Meta의 Brain2Qwerty v2([6월 말 발표](https://ai.meta.com/blog/brain2qwerty-brain-ai-human-communication/)) — 에이전트가 코드베이스를 반복 수정해 더 나은 디코딩 아키텍처를 발명했고 단어 오류율(word error rate)을 크게 개선했다. 단, 최종 학습 설정은 여전히 사람이 손으로 골랐다 — 플래그십 시스템 루프도 마지막 체크포인트엔 사람을 둔다.
- **종료 신호**: 네 루프 중 가장 까다로운 신호 집합 — 평가, 저지(judge), 필터링된 제품 피드백, 그리고 Roland의 설계에서는 에이전트가 신입 사원처럼 암묵지를 축적하는 명시적 ask-a-human 도구.

## Agentic MapReduce는 루프인가?

같은 주에 유명해진 Cognition의 [Devin Security Swarm](https://cognition.com/blog/introducing-devin-security-swarm)은 경계가 정해진 병렬 에이전트들을 저장소에 흩뿌리고 결과를 취합하는 패턴(Agentic MapReduce)으로, 루프라 불리지만 저자는 루프가 아니라고 본다. 분배-수집-검증은 파이프라인이다: 아무것도 다음 사이클로 피드백되지 않고, **피드백 없는 루프는 그냥 for 문**이다. 팬아웃은 네 루프 중 어디에나 배치할 수 있는 토폴로지이지, 그 자체로 루프가 아니다.

## 이름 없는 맨 위 루프 = 오버사이트 루프

swyx의 다이어그램에서 루프를 만드는 루프 위 가장 바깥 고리는 문자 그대로 "???? loop"로 표기돼 있다. 그 동사는 목표 설정, 자원 배분, 솎아내기(cull)이고 종료 조건은 "없음"으로 적혀 있다. 저자는 이를 **오버사이트 루프(oversight loop)** 라 명명한다: 목표가 정해지고 예산이 배분되고 작업이 걸러지는 곳, 그리고 사람이 살아야 할 유일한 고리다. Addy가 AIEWF 무대에서 말했듯 "내부 루프는 능력(capability)이고 외부 루프는 주체성(agency)이다."

AIEWF의 가장 날카로운 논쟁들은 번역해 보면 모두 그 맨 위 고리를 누가 운영하느냐에 대한 것이었다.

- **다이얼을 올리자는 쪽**: Zach와 Roland — 체크포인트를 의도적으로 고르고, 신뢰가 쌓이는 만큼 자율성을 단계적으로 올린다. Roland의 인상적인 구분: 팩토리보다 먼저 오케스트라를 지어라(오케스트라는 인간 지휘자를 유지하는 시스템).
- **다이얼에 한계가 있다는 쪽**: Notion의 Geoffrey Litt는 팩토리를 우울한 비전이라 부르며 [에세이](https://www.geoffreylitt.com/2026/07/02/understanding-is-the-new-bottleneck.html)에서 이해를 위임하는 자는 에이전트에게 대체된다고 주장했다. Paul Bakaus는 단호하게 ["There is no auto, and there will be no auto"](https://www.latent.space/p/skill-engineering-design)라 했다 — 품질만이 아니라 소유(ownership)의 문제이며, 사람에겐 목적이 필요하고 자신이 만드는 것에서 역할을 원한다.
- 폐막 토론: HumanLayer의 Dex Horthy는 반(反)루프가 아니라며 Kubernetes도 컨트롤 루프 위에 지어졌지만 결정론적 루프라 지적했고, 열광이 엔지니어링을 앞질렀다고 걱정하며 추상화 수준을 올리지 말고 한 단계 내려가라고 조언했다. Geoffrey는 반대편에서 루프는 불가피하다고 했다. Mike는 가장 솔직한 데이터를 내놨다: Anthropic 내부에서도 Tag를 운영하는 팀이 리뷰와, 시스템이 뭘 하는지 개념화하는 인간의 능력에서 병목을 겪는다 — 사람이 스스로에게 남겨 둔 체크포인트가 이제 제약이 됐다.

자율성은 네 루프 각각에 별도로 존재하는 다이얼이다. 강하게 감독되는 프로덕트 루프 안에서 완전 자율 실행 루프를 돌릴 수도 있고, 목표 설정은 전적으로 사람이 하면서 시스템 루프를 에이전트에게 넘길 수도 있다. 흥미로운 엔지니어링 질문은 어느 진영이 이기느냐가 아니라, 각 다이얼을 올바르게 설정하려면 어떤 정보가 필요한가다. 모든 루프에는 — 맨 위 것을 포함해 — 이름 붙일 수 있는 종료 조건이 있고, 맨 위 루프의 종료 조건은 바로 당신이다. 다만 신호에 이름을 붙이는 것과 그것을 실제로 연결하는 것은 다르다: 신호 없는 루프는 수렴하지 않고 외부의 무언가가 멈출 때까지 그냥 돈다. 프로덕션 규모에서 루프가 실제로 닫히고 있는지 알려면 트랜스크립트를 표본 검사하는 대신 트레이스를 지속적으로 훑고 실패를 클러스터링해야 하며, 그것이 바로 [Arize AX](https://arize.com/)가 만들어진 목적이다.

## 당신은 어느 루프를 만들고 있는가

루프에 이름이 붙었으니 그것이 물어야 할 질문이다. 네 루프 모두의 밑에는 같은 실천이 깔려 있다: 사람들은 추상화 수준을 올리고 인간의 판단을 스택 위쪽으로 밀어 올리고 있다. 그것이 루프의 진짜 교훈이다 — 스택을 올라가며 더 많은 것을 해내고, 이제 지도가 있으니 어디로 올라가야 할지 안다. 이 글은 [@seldo](https://x.com/@seldo)와 공동 저술됐다.

## 관련 노트

- [[art-of-loop-engineering]] — 본문이 인용한 LangChain의 4단계 루프 스택 정리
- [[getting-started-with-loops]] — Claude Code 팀이 정의하는 4가지 루프의 실천편
- [[Loop Engineering|루프 엔지니어링]] — Addy Osmani가 제시한 루프 엔지니어링 개념 용어

> 원문: [What the hell is a loop, anyway?](https://x.com/aparnadhinak/status/2073492320159510869)
> 원본 클립: [[2026-07-14-What the hell is a loop, anyway?]]
