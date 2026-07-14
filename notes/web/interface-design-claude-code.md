---
title: "interface-design — Claude Code용 디자인 엔지니어링 스킬 (일관된 UI를 위한 크래프트·메모리·강제)"
date: 2026-07-14 14:24:09 +0900
categories: [개발, 도구]
tags: [claude-code, codex, skill, design-system, ui, design-engineering]
slug: interface-design-claude-code
publish: true
---
Dammyjay93/interface-design는 코딩 에이전트로 UI를 만들 때 세션마다 흐트러지는 디자인 결정(간격, 색, 깊이 전략, 표면 엘리베이션)을 `.interface-design/system.md`에 저장해 두고 다시 불러와, 세션을 넘나들며 일관된 인터페이스를 유지하게 해주는 에이전트 스킬이다. 대시보드·앱·툴·관리자 패널 같은 인터페이스 디자인용이며, 마케팅 사이트용은 아니다.

## 무엇을 해주나

- **크래프트(Craft)** — 원칙 기반 디자인으로 진짜 시각적 위계가 있는, 전문적이고 다듬어진 인터페이스를 만든다.
- **비주얼 디렉션** — 이미지 생성 도구가 있으면 디렉션 보드, UI 레퍼런스, 페인트오버(paintover) 크리틱을 탐색한다.
- **메모리(Memory)** — 결정을 `.interface-design/system.md`에 저장하고, 스킬 실행 시 다시 로드한다.
- **일관성(Consistency)** — 세션 내내 UI 변경이 같은 원칙을 따른다. 선택은 한 번, 적용은 일관되게.

### 도입 전후 비교

- 없이: 세션마다 처음부터 시작, 버튼 높이가 36/38/40px로 떠다니고, 간격 값이 14/17/22px로 제멋대로, 컴포넌트 간 일관성 없음.
- 있으면: 스킬 실행 시 시스템 자동 로드, 패턴 재사용(Button 36px, Card 16px 패딩), 그리드에 맞는 간격(4/8/12/16px), 일관된 깊이·표면 처리.

## 설치

권장은 skills.sh CLI다. CLI가 지원 에이전트를 감지해 알맞은 위치에 설치한다:

```
npx skills add https://github.com/dammyjay93/interface-design --skill interface-design
```

- 전역 설치: `-g`, 특정 에이전트 지정: `--agent claude-code -g` 또는 `--agent codex -g`, 모든 에이전트: `--agent '*' -g -y`, 설치 없이 목록 보기: `--list`.
- **Claude Code**: `~/.claude/skills/interface-design`에 설치되고, 자동 호출되거나 `/interface-design`으로 직접 호출한다. 기존 플러그인 마켓플레이스 방식(`/plugin marketplace add Dammyjay93/interface-design` → `/plugin menu`)도 여전히 지원.
- **Codex**: `~/.agents/skills/interface-design`에 설치. 스킬이 바로 안 보이면 Codex 재시작 또는 새 스레드.
- **수동 설치**: 저장소 클론 후 `.claude/skills/interface-design` 폴더를 `~/.claude/skills/`(Claude Code) 또는 `~/.agents/skills/`(Codex)로 복사하고 에이전트 재시작.
- 설치한 스킬은 코딩 에이전트와 같은 권한으로 실행되므로 사용 전 검토할 것.

## 동작 방식

- **system.md가 있으면**: 스킬 파일·원칙을 읽고 → `.interface-design/system.md` 로드 → 확립된 패턴 적용 → 디자인 선택을 명시적으로 유지 → 새 패턴 저장 제안.
- **없으면**: 원칙을 읽고 → 프로젝트 컨텍스트 평가 → 방향 제안 후 확인 요청 → 명시적 선택을 유지하며 일관된 원칙으로 빌드 → 시스템 저장 제안.
- 예시 흐름: "메트릭 카드가 있는 사용자 대시보드 만들어줘" → 에이전트가 "데이터 중심 대시보드니 Depth: 보더 온리, Surfaces: 미묘한 엘리베이션, Spacing: 8px 베이스" 방향을 제안 → 승인하면 그 기준으로 빌드 → system.md 저장 제안. 다음 세션에 "설정 페이지 추가해줘"라고 하면 system.md를 로드해 기존 시스템에 맞춰 만든다. 시스템이 세션을 넘어 **기억**된다.

## 시스템 파일 (system.md)

방향이 정해지면 결정이 `.interface-design/system.md`에 산다. 예: Direction(Personality: Precision & Density, Foundation: Cool(slate), Depth: Borders-only), Tokens(Spacing base 4px / 스케일 4~32, 색상 변수), Patterns(Button Primary: 높이 36px·패딩 12px 16px·radius 6px, Card Default: 0.5px 보더·패딩 16px·radius 8px). 이 파일을 Claude Code와 Codex가 읽어 일관성을 유지한다.

## 사용법과 리뷰 커맨드

- 기본 호출: `/interface-design`, `use interface-design to build this dashboard`, `use interface-design to audit this settings page`.
- Claude Code 플러그인 설치 시 `.claude/commands`에 리뷰용 커맨드 2개가 생긴다:
  - `/interface-design:design-review` — 엄격한 크래프트·위계 리뷰(승인 기준선 포함). 포컬 포인트, 위계, 타이포그래피, 색, 표면, 상태, 모션을 최상급 디자인 팀의 기준으로 판정하고 고칠 점을 알려주는, 딥 코드 리뷰의 디자인판.
  - `/interface-design:design-deslop` — 브랜치 diff 범위에서 생성형 UI 특유의 티(제네릭 토큰, 기본 폰트, 소심한 팔레트, 똑같은 카드, 누락된 상태, 거친 보더)를 찾아 제거하는 빠른 패스.
- Codex도 같은 스킬 콘텐츠를 슬래시 커맨드나 자연어로 호출한다. 이미지 생성 도구가 있으면 디렉션 보드, UI 레퍼런스 목업, 스크린샷 페인트오버, 프로젝트 결속 래스터 에셋을 만들 수 있다.

## 디자인 디렉션 6종

스킬이 프로젝트 컨텍스트에서 방향을 추론하지만 커스터마이즈도 가능하다.

| 디렉션 | 느낌 | 적합한 곳 |
| --- | --- | --- |
| Precision & Density | 타이트, 기술적, 모노크롬 | 개발자 도구, 관리자 대시보드 |
| Warmth & Approachability | 넉넉한 간격, 부드러운 그림자 | 협업 도구, 컨슈머 앱 |
| Sophistication & Trust | 쿨 톤, 레이어드 깊이 | 금융, 엔터프라이즈 B2B |
| Boldness & Clarity | 높은 대비, 극적인 여백 | 모던 대시보드, 데이터 헤비 앱 |
| Utility & Function | 차분하고 기능적인 밀도 | GitHub 스타일 도구 |
| Data & Analysis | 차트 최적화, 숫자 우선 | 애널리틱스, BI 도구 |

시스템 파일 템플릿은 `reference/examples/`의 system-precision.md(대시보드/관리자용), system-warmth.md(협업/컨슈머용) 참고.

## 기타

- 이 저장소는 `claude-design-skill`에서 이름이 바뀌었고 옛 URL은 자동 리다이렉트된다. 옛 스킬(`~/.claude/skills/design-principles`)을 지우고 새로 설치하면 되며, 기존 system.md는 그대로 동작한다(`.ds-engineer/`는 `.interface-design/`로 이름 변경).
- 철학: **결정은 복리로 쌓인다**(한 번 정한 간격 값이 패턴이 되고, 깊이 전략이 아이덴티티가 된다). **일관성이 완벽함을 이긴다**("불완전한" 값의 일관된 시스템이 "올바른" 값의 산만한 인터페이스보다 낫다). **메모리가 반복(iteration)을 가능케 한다**(무엇을 왜 결정했는지 보이면 우연히 표류하는 대신 의도적으로 진화할 수 있다).
- 라이선스: MIT.

## 관련 노트

- [[emilkowalski-design-skills]] — 같은 디자인 엔지니어링 계열의 AI 에이전트 스킬
- [[디자인 시스템 프롬프트 모음]] — 디자인 시스템 구축 프롬프트 참고
- [[alirezarezvani-claude-skills]] — 345+ 스킬 라이브러리(UI 디자인 스킬 포함)
- [[mattpocock-skills]] — 코딩 에이전트용 스킬 모음

> 원문: [Dammyjay93/interface-design: Design engineering for Claude Code. Craft, memory, and enforcement for consistent UI.](https://github.com/Dammyjay93/interface-design)
> 원본 클립: [[2026-07-14-Dammyjay93interface-design Design engineering for Claude Code. Craft, memory, and enforcement for consistent UI.]]
