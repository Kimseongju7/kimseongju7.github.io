---
title: "prompt-master — 어떤 AI 도구용 프롬프트든 정확하게 써주는 Claude 스킬"
date: 2026-07-14 14:24:45 +0900
categories: [개발, 도구]
tags: [claude, skill, prompt-engineering, llm, ai, claude-code]
slug: prompt-master-skill
publish: true
---
nidhinjs/prompt-master는 어떤 AI 도구를 쓰든 그 도구에 맞는 정확한 프롬프트를 대신 써주는 Claude 스킬이다. 토큰·크레딧 낭비 없이, 컨텍스트와 메모리를 온전히 유지해, 첫 시도에 받았어야 할 답을 재프롬프트로 더듬어 찾는 일을 없애는 것이 목표다. Claude, ChatGPT, Gemini, o1/o3, MiniMax, Cursor, Claude Code, GitHub Copilot, Windsurf, Bolt, v0, Lovable, Devin, Perplexity, Midjourney, DALL-E, Stable Diffusion, ComfyUI, Sora, Runway, ElevenLabs, Zapier, Make 등 사실상 모든 AI 도구와 함께 쓸 수 있다.

## 해결하는 문제

- 모든 AI 사용자가 같은 방식으로 크레딧을 낭비한다: 애매한 프롬프트 작성 → 잘못된 출력 → 재프롬프트 → 조금 개선 → 또 재프롬프트 → 4번째 시도에야 원하는 결과. API 호출 3번이 낭비되고, 하루 50개 프롬프트면 실제 돈과 시간이 사라진다.
- 핵심 통찰: **"최고의 프롬프트는 가장 긴 프롬프트가 아니라, 모든 단어가 하중을 받는(load-bearing) 프롬프트다."** 대부분의 "프롬프트 생성기"는 프롬프트를 길게 만들지만, 이 스킬은 더 날카롭게 만든다.

## 설치와 사용

- 설치: 저장소를 ZIP으로 받아 claude.ai → Sidebar → Customize → Skills → Upload a Skill. 또는(비권장) `git clone https://github.com/nidhinjs/prompt-master.git ~/.claude/skills/prompt-master`.
- 자연어로 호출: "Cursor용으로 auth 모듈 리팩터링 프롬프트 써줘", "Claude Code용 REST API 빌드 프롬프트가 필요해 — 필요한 걸 물어봐", "GPT-4o용으로 쓴 나쁜 프롬프트 고쳐줘", "사이버펑크 도시 야경 Midjourney 프롬프트 만들어줘", "이 프롬프트를 Stable Diffusion용으로 분해·변환해줘" 등.
- 명시적 호출: `/prompt-master` 뒤에 요청을 붙인다.

## 동작 파이프라인 (7단계)

1. **타깃 도구 감지** — 어느 AI 시스템용 프롬프트인지 파악하고 조용히 알맞은 접근으로 라우팅
2. **의도 9차원 추출** — 작업, 입력, 출력, 제약, 컨텍스트, 청중, 메모리, 성공 기준, 예시
3. **핵심 정보가 빠졌을 때만 타깃 질문** — 최대 3개, 그 이상은 절대 안 함
4. **알맞은 프레임워크로 라우팅** — 올바른 프롬프트 아키텍처를 자동 선택·적용(사용자에게는 안 보임)
5. **안전한 기법만 적용** — 역할 부여, few-shot 예시, XML 구조, 그라운딩 앵커, 메모리 블록을 필요 시에만
6. **토큰 효율 감사** — 출력을 바꾸지 않는 단어를 전부 제거
7. **프롬프트 전달** — 복사 가능한 깔끔한 블록 하나 + 한 줄 전략 노트

## 예시

- **이미지**: "밤비 속 사무라이 Midjourney 프롬프트" 요청 → 콤마 구분 묘사어, 조명·무드 앞배치, `--ar 16:9 --v 6 --style raw` 파라미터 고정, 스타일 표류를 막는 네거티브 프롬프트까지 포함된 약 60토큰짜리 프롬프트를 생성.
- **코딩**: "Notion 느낌의 비즈니스 대시보드 랜딩 페이지를 Claude Code로" 요청 → 애매한 "Notion 미학"을 정확한 hex 값·픽셀 스펙(색상, Inter 폰트, 8px 간격, radius, 그림자)으로 번역하고, 8개 섹션의 빌드 순서, IntersectionObserver 기반 애니메이션의 정확한 타이밍·방식·트리거, 단일 파일 제약, "Done When" 완료 기준까지 명시한 약 380토큰짜리 프롬프트를 생성. Claude Code가 잘못 추측할 여지를 없앤다.

## 도구 프로필과 템플릿

- **30개 이상의 도구 프로필** 내장: 도구 카테고리별로 고치는 지점이 다르다. 예를 들어 Claude는 패딩 제거 + XML 구조, o3/o4-mini 같은 추론 모델에는 CoT를 절대 추가하지 않고(내부적으로 사고하므로) 짧고 깨끗한 지시만, Claude Code는 정지 조건·파일 범위·체크포인트 출력, Cursor/Windsurf는 파일 경로·함수명·건드리면 안 되는 목록, Midjourney는 콤마 구분 묘사어와 네거티브 프롬프트, Stable Diffusion은 `(word:1.3)` 가중치 문법과 필수 네거티브 프롬프트, Zapier/Make/n8n은 트리거 앱+이벤트와 액션 필드 매핑 등.
- 목록에 없는 도구는 **Universal Fingerprint**(4가지 질문)로 처음 보는 AI 시스템용 프롬프트도 품질 있게 작성한다.
- **12개 프롬프트 템플릿 자동 선택**: RTF(빠른 원샷), CO-STAR(비즈니스 문서), RISEN(복잡한 다단계 프로젝트), CRISPE(크리에이티브), Chain of Thought(수학·로직·디버깅), Few-Shot(일관된 구조화 출력), File-Scope(코드 편집 AI), ReAct+정지 조건(자율 에이전트), Visual Descriptor(이미지·영상 생성), Reference Image Editing(편집 vs 생성 자동 감지), ComfyUI(노드 기반 워크플로), Prompt Decompiler(기존 프롬프트 분해·변환). 프레임워크 이름은 사용자에게 보여주지 않고 조용히 라우팅한다.

## 안전한 기법 5가지만 사용

신뢰할 수 있고 효과가 한정된 기법만 쓴다. 환각이나 예측 불가 출력을 만드는 것으로 알려진 기법(Tree of Thought, Graph of Thought, Universal Self-Consistency, prompt chaining)은 명시적으로 배제한다.

| 기법 | 하는 일 |
| --- | --- |
| 역할 부여 | 특정 전문가 정체성을 부여해 깊이와 어휘를 보정 |
| Few-Shot 예시 | 형식 일관성이 중요할 때 2~5개 예시 추가 |
| XML 구조 태그 | XML을 안정적으로 파싱하는 Claude 계열 도구용 |
| 그라운딩 앵커 | 사실·인용 작업에 반환각(anti-hallucination) 규칙 추가 |
| Chain of Thought | 로직 작업에 단계별 추론 강제 — o1/o3에는 절대 적용 안 함 |

## 35가지 크레딧 낭비 패턴 감지

Before/After 예시와 함께 6개 카테고리로 감지·교정한다.

- **작업 패턴(7)**: 애매한 동사("help me with my code" → "`getUserData()`를 async/await로 리팩터링하고 null 반환 처리"), 한 프롬프트에 두 작업, 성공 기준 부재, 과잉 허용 에이전트, 감정적 서술, 통짜 앱 빌드 요구, 암묵적 참조("아까 얘기한 거") 등.
- **컨텍스트 패턴(6)**: 사전 지식 가정("이어서 해줘" → 메모리 블록 포함), 프로젝트 컨텍스트 부재, 잊힌 스택, 환각 유도 질문("전문가들은 뭐라고 해?" → "확실한 출처만 인용, 불확실하면 말하라"), 청중 미정의, 이전 실패 미언급.
- **형식 패턴(6)**: 출력 형식·길이 미지정, 역할 미부여, 애매한 미적 형용사("professional하게" → 구체적 팔레트·폰트·행간 스펙), 이미지 AI에 네거티브 프롬프트 누락, Midjourney에 산문 프롬프트 사용.
- **범위 패턴(6)**: 범위 경계 없음("앱 고쳐줘" → "`src/auth.js`의 로그인 폼 검증만 수정, 나머지 금지"), 스택 제약 없음, 에이전트 정지 조건 없음, IDE AI에 파일 경로 없음, 도구에 안 맞는 템플릿, 코드베이스 통째 붙여넣기.
- **추론 패턴(5)**: 로직 작업에 CoT 누락, 추론 모델에 CoT 추가(출력 저하), 복잡한 출력에 자기 검증 없음, 세션 간 기억 기대, 이전 결정과 모순.
- **에이전트 패턴(5)**: 시작 상태·목표 상태 미지정, 침묵하는 에이전트(단계별 진행 출력 요구), 파일시스템 무제한(편집 가능 경로 제한), 사람 리뷰 트리거 없음(파일 삭제·의존성 추가·DB 스키마 변경 전 정지·질문).

## 메모리 블록 시스템

대화에 히스토리가 있으면 이전 결정들을 뽑아 메모리 블록(예: 스택은 React 18 + TypeScript + Supabase, 인증은 httpOnly 쿠키의 JWT, 컴포넌트 네이밍은 PascalCase, Tailwind만 사용, Redux 없이 Context API만)을 프롬프트 앞에 붙여, AI가 이전 작업과 모순되지 않게 한다. 긴 세션에서 낭비되는 재프롬프트 대부분이 AI가 이미 결정한 걸 잊는 데서 오므로, 이것이 가장 큰 단일 개선이다.

## 버전 히스토리 (요약)

- 1.7.0 — Opus 4.8 호환. Claude 4.x 라우팅을 버전 인지형으로(4.6/4.7/4.8 일반화), Opus 4.8 프로필 추가.
- 1.6.0 — Opus 4.7 대응. Template M(Opus 4.7 Task Brief), 패턴 36~37 추가.
- 1.5.0 — Agentic AI·3D 모델 AI 라우팅 추가.
- 1.4.0 — 레퍼런스 이미지 편집 감지, ComfyUI 지원, Prompt Decompiler 모드.
- 1.3.0 — PAC2026 위치 구조(30/55/15) 기반 재구축, 사일런트 라우팅 도입.
- 1.2.0 — 어텐션 아키텍처 재구성, 환각 유발 기법(ToT, GoT, USC, 체이닝) 제거.
- 1.1.0 — 도구 커버리지 확장, 메모리 블록 시스템·35개 패턴 추가.
- 라이선스: MIT.

## 관련 노트

- [[alirezarezvani-claude-skills]] — 대규모 스킬 라이브러리(프롬프트 템플릿 등 포함)
- [[prompting-claude-fable-5]] — Claude 계열 프롬프팅 기법
- [[claude-code-model-and-effort]] — 토큰 효율·모델별 특성 이해
- [[midudev-autoskills]] — 스킬 자동 생성

> 원문: [nidhinjs/prompt-master: A Claude skill that writes the accurate prompts for any AI tool. Zero tokens or credits wasted. Full context and memory retention](https://github.com/nidhinjs/prompt-master)
> 원본 클립: [[2026-07-14-nidhinjsprompt-master A Claude skill that writes the accurate prompts for any AI tool. Zero tokens or credits wasted. Full context and memory retention]]
