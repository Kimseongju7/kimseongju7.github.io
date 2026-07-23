---
title: "DietrichGebert/ponytail — AI 에이전트를 '가장 게으른 시니어 개발자'로 만드는 플러그인"
date: 2026-07-14 14:19:23 +0900
categories: [개발, 도구]
tags: [claude-code, codex, plugin, yagni, ai-agent, prompt, benchmark]
slug: ponytail-lazy-senior-dev
publish: true
---
**Ponytail**은 AI 코딩 에이전트가 "방에서 가장 게으른 시니어 개발자"처럼 생각하게 만드는 플러그인이다. 50줄을 보여주면 말없이 한 줄로 바꿔놓는 그 시니어를 에이전트 안에 넣는다는 컨셉으로, 실측 기준 **코드 약 54% 감소(최대 94%) · 비용 약 20% 절감 · 속도 약 27% 향상 · 안전성 100% 유지**를 내세운다.

## Before / After

날짜 선택기를 요청하면 보통 에이전트는 flatpickr를 설치하고, 래퍼 컴포넌트를 쓰고, 스타일시트를 추가하고, 타임존 논의까지 시작한다. ponytail을 쓰면:

```
<!-- ponytail: browser has one -->
<input type="date">
```

## 벤치마크 수치

정직한 측정은 실제 에이전트의 실제 작업이라는 입장이다. 헤드리스 Claude Code 세션이 실제 FastAPI + React 저장소(tiangolo의 full-stack-fastapi-template)를 편집하고, 남긴 `git diff`로 채점했다. 피처 티켓 12개, 스킬 유무 동일 에이전트, n=4, Haiku 4.5.

| 무스킬 베이스라인 대비 | LOC | 토큰 | 비용 | 시간 | 안전 |
| --- | --- | --- | --- | --- | --- |
| **ponytail** | **-54%** | **-22%** | **-20%** | **-27%** | **100%** |
| caveman (간결-산문 대조군) | -20% | +7% | +3% | +2% | 100% |
| "[[YAGNI]] + 원라이너" 프롬프트 | -33% | -14% | -21% | -30% | 95% |

ponytail은 모든 지표를 줄인 유일한 조건이자, 그러면서도 완전히 안전한 유일한 조건이다. 감소 폭은 실제 과잉 구축 함정이 있는 곳에서 가장 크고(날짜 선택기 404줄→23줄, 컬러 피커 287줄→23줄 — 컴포넌트 대신 네이티브 `<input>`을 쓰기 때문), 이미 최소인 코드에서는 거의 0이다.

이전 단발(single-shot) 벤치마크는 80-94% 감소를 보고했지만, 이슈 #126이 지적했듯 베어 모델 베이스라인이 산문과 선택지로 답을 부풀리는 대화형 베이스라인 아티팩트가 섞여 있었다. 공정한 에이전트 베이스라인 대비로는 94%가 태스크별 상한이지 평균이 아니며, 위의 에이전트 수치가 수정된 방어 가능한 버전이다.

규칙은 "최소 토큰"이 아니라 **"작업에 필요한 것만 쓰되, 검증·에러 처리·보안·접근성은 절대 자르지 않는다"** 이다. 코드가 작아지는 건 골프가 아니라 필요한 것만 남아서다. 비용·지연 감소는 사다리를 따르는 모델에서의 부수 효과이며, 단계를 따지느라 사고 토큰을 쓰는 간결한 추론 모델(GPT-5.5)에서는 반대로 갈 수도 있다.

## 동작 원리: 사다리(ladder)

코드를 쓰기 전에 에이전트는 처음으로 성립하는 단(rung)에서 멈춘다.

```
1. 이게 존재할 필요가 있나?     → 없으면 건너뜀 (YAGNI)
2. 이 코드베이스에 이미 있나?   → 재사용, 다시 쓰지 않음
3. 표준 라이브러리가 해주나?    → 그걸 사용
4. 네이티브 플랫폼 기능인가?    → 그걸 사용
5. 설치된 의존성이 해주나?      → 그걸 사용
6. 한 줄이면 되나?              → 한 줄
7. 그제서야: 동작하는 최소한
```

사다리는 문제를 이해한 *뒤에* 돌지, 이해를 대신하지 않는다. 변경이 닿는 코드를 읽고 실제 흐름을 추적한 뒤에 단을 고른다. **해법에 게으르되, 읽기에 게으르지 않다.** 그리고 게으르지만 태만하지 않다: 신뢰 경계 검증, 데이터 손실 처리, 보안, 접근성은 절대 도마 위에 오르지 않는다.

## 설치

Claude Code와 Codex 플러그인은 작은 Node.js 라이프사이클 훅 2개를 돌리므로 `node`가 PATH에 있어야 한다(Nix/nvm 사용자는 비대화형 셸 PATH 주의). 없어도 스킬은 동작하고, 상시 활성화만 조용히 꺼진다.

- **Claude Code**: `/plugin marketplace add DietrichGebert/ponytail` → `/plugin install ponytail@ponytail` (두 개를 별도 프롬프트로 보내야 함)
- **Codex**: `codex plugin marketplace add DietrichGebert/ponytail` → `codex plugin add ponytail@ponytail`, `/hooks`에서 훅 2개 신뢰 후 새 스레드
- **GitHub Copilot CLI**: `copilot plugin marketplace add ...` / `copilot plugin install ...`, 명령은 `/ponytail:ponytail`처럼 플러그인 이름으로 네임스페이스됨
- 그 외 Pi, OpenCode, Gemini CLI, Qoder, Antigravity CLI, Hermes Agent, CodeWhale, Swival, Devin CLI, OpenClaw 등 방대한 호스트별 설치 경로를 지원한다.
- Cursor, Windsurf, Cline, Copilot Chat, Aider, Kiro, Zed 등 명령이 안 되는 호스트는 저장소의 해당 규칙 파일(`.cursor/rules/`, `AGENTS.md` 등)을 복사하면 상시 규칙만 적용된다.

기본 레벨은 `PONYTAIL_DEFAULT_MODE` 환경 변수(`lite`/`full`/`ultra`/`off`) 또는 `~/.config/ponytail/config.json`의 `defaultMode`로 설정하며, 기본값은 `full`이다. 활성 중에는 Agent 툴로 스폰되는 모든 서브에이전트에도 규칙이 주입되며, `PONYTAIL_SUBAGENT_MATCHER` 정규식으로 특정 에이전트 타입에만 제한할 수 있다.

## 명령어

| 명령 | 하는 일 |
| --- | --- |
| `/ponytail [lite\|full\|ultra\|off]` | 강도 설정 또는 끄기. 인자 없으면 현재 레벨 보고 |
| `/ponytail-review` | 현재 diff의 과잉 엔지니어링을 리뷰하고 삭제 목록을 반환 |
| `/ponytail-audit` | diff가 아니라 저장소 전체의 과잉 엔지니어링 감사 |
| `/ponytail-debt` | 미뤄둔 `ponytail:` 단축들을 장부로 수확 — "나중에"가 "안 함"이 되지 않게 |
| `/ponytail-gain` | 벤치마크의 측정된 임팩트 스코어보드 표시 |
| `/ponytail-help` | 명령 빠른 참조 |

`/ponytail ultra`는 "코드베이스가 당신에게 개인적으로 잘못했을 때"를 위한 모드다. 명령은 스킬 가능 호스트(Claude Code, Codex, Devin CLI, OpenCode, Gemini, pi, Swival, Hermes, Qoder)가 필요하고, Codex에서는 `@ponytail-review`처럼 `@`로 호출한다.

## 제거와 FAQ

호스트별 제거 명령(`/plugin remove ponytail` 등) 후에도 모드 플래그, `~/.config/ponytail/config.json`, (셋업 시 수락했다면) `~/.claude/settings.json`의 statusLine 항목이 남는데 `node scripts/uninstall.js`로 정리한다 — 스크립트 자체가 플러그인 파일이므로 **호스트 제거 명령보다 먼저 실행**해야 한다.

- **설정 파일 필요?** 아니오. 전부 선택 사항.
- **정말 120줄짜리 캐시 클래스가 필요하다면?** 필요 없다. 그래도 우기면 만들어 준다. 천천히. 정확하게. 당신을 쳐다보면서.
- **스케일 되나?** 안 쓴 코드는 무한히 스케일된다. 버그 0, CVE 0, 가동률 100%.

라이선스는 MIT — "동작하는 가장 짧은 라이선스".

## 관련 노트

- [[harness-engineering-codex]] — "황금 원칙"·가비지 컬렉션 등 최소 코드·드리프트 억제 철학과 통함
- [[claude-code-model-and-effort]] — 모델·effort에 따라 토큰 절감이 달라지는 벤치마크 맥락
- [[YAGNI]] — 사다리 1단 "이게 존재할 필요가 있나?"의 근간이 되는 설계 원칙

> 원문: [DietrichGebert/ponytail: Makes your AI agent think like the laziest senior dev in the room.](https://github.com/DietrichGebert/ponytail)
> 원본 클립: [[2026-07-14-DietrichGebertponytail Makes your AI agent think like the laziest senior dev in the room. The best code is the code you never wrote.]]
