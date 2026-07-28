---
title: 'Caveman — 원시인처럼 말해서 출력 토큰 65%를 줄이는 코딩 에이전트 스킬'
date: 2026-07-26 16:54:38 +0900
categories: [개발, 도구]
tags: [caveman, claude-code, tokens, prompt-engineering, ai-agent, skills, open-source]
description: 'Caveman은 AI 코딩 에이전트가 원시인처럼 짧게 말하게 만드는 스킬/플러그인이다. 답의 내용은 그대로 두고 군더더기만 걷어내 출력 토큰을 평균 65% 줄이며, 코드·명령어·에러 메시지는 바이트 단위로 그대로 보존한다. "b…'
---
**Caveman**은 AI 코딩 에이전트가 원시인처럼 짧게 말하게 만드는 스킬/플러그인이다. 답의 내용은 그대로 두고 군더더기만 걷어내 **출력 토큰을 평균 65% 줄이며**, 코드·명령어·에러 메시지는 바이트 단위로 그대로 보존한다. "brain still big, mouth small" — 뇌를 줄이는 게 아니라 입을 줄인다.

## 무엇을 하는가

Claude Code, Codex, Gemini, Cursor, Windsurf, Cline, Copilot 등 30개 이상 에이전트에서 동작한다. 한 번 설치하면 에이전트가 filler를 버리고 압축된 caveman 말투로 답하되, 코드·명령어·에러는 원문 그대로 유지한다. 이후 모든 응답에서 출력 토큰이 절약된다.

### Before / After

| 일반 에이전트 — 69 토큰 | Caveman 에이전트 — 19 토큰 |
| --- | --- |
| > React 컴포넌트가 리렌더되는 이유는 렌더 사이클마다 새 객체 참조를 만들고 있기 때문일 가능성이 큽니다. 인라인 객체를 prop으로 넘기면 React의 얕은 비교가 매번 다른 객체로 인식해서 리렌더를 유발합니다. useMemo로 객체를 메모이제이션하는 것을 권장드립니다. | > New object ref each render. Inline object prop = new ref = re-render. Wrap in `useMemo`. |
| > 물론이죠! 기꺼이 도와드리겠습니다. 겪고 계신 문제는 인증 미들웨어가 토큰 만료를 제대로 검증하지 않아서 발생하는 것으로 보입니다. 살펴보고 수정안을 제안드리겠습니다. | > Bug in auth middleware. Token expiry check use `<` not `<=`. Fix: |

같은 수정, 3분의 1의 단어. 기술적 내용은 하나도 잃지 않는다.

```
┌────────────────────────────────────────────┐
│   output tokens saved   █████████       65% │
│   input tokens saved    ░░░░░░░░░         0% │
│   technical accuracy    █████████      100% │
│   vibes                 █████████       OOG │
└────────────────────────────────────────────┘
```

## 설치

명령 하나로 머신에 있는 모든 에이전트를 찾아 각각 설치한다. 약 30초, Node ≥18 필요. 없는 에이전트는 건너뛰고, 재실행해도 안전하다.

```
# macOS · Linux · WSL · Git Bash
curl -fsSL https://raw.githubusercontent.com/JuliusBrussee/caveman/main/install.sh | bash
```
```
# Windows · PowerShell 5.1+
irm https://raw.githubusercontent.com/JuliusBrussee/caveman/main/install.ps1 | iex
```

켜기는 `/caveman` 입력 또는 *"talk like caveman"*, 끄기는 *"normal mode"*. Claude Code·Codex·Gemini에서는 첫 메시지부터 이미 켜져 있어 별도 명령이 필요 없다.

에이전트별로 경로(플러그인, 익스텐션, 룰 파일, `npx skills add`)가 다르며 전체 매트릭스는 `INSTALL.md`에 있다. 자주 쓰는 것들:

```
# Claude Code plugin
claude plugin marketplace add JuliusBrussee/caveman && claude plugin install caveman@caveman

# Gemini CLI extension
gemini extensions install https://github.com/JuliusBrussee/caveman

# Cursor / Windsurf / Cline / Codex / 30+ more, via the skills registry
npx skills add JuliusBrussee/caveman -a cursor
```

설치가 깨지면 이 저장소에서 에이전트를 열고 *"Read CLAUDE.md and INSTALL.md, install caveman for me."*라고 시키면 된다.

## 압축 레벨

여섯 단계가 있고 `/caveman <level>`로 언제든 바꾼다. 레벨은 바꾸거나 세션이 끝날 때까지 유지된다.

| 레벨 | 같은 문장, 줄어든 형태 |
| --- | --- |
| *normal agent* | 렌더마다 새 참조가 생기므로 객체를 `useMemo`로 감싸야 합니다. |
| `lite` | Wrap object in `useMemo`. New ref created every render. |
| `full` *(기본값)* | New ref each render. Wrap object in `useMemo`. |
| `ultra` | New ref/render. `useMemo` it. |
| `wenyan` | 렌더마다 새 참조가 생기니 `useMemo`로 감싸라 — 고전 중국어(문언문)로 표현해 더 짧다. |

Caveman은 사용자의 언어를 유지한다. 포르투갈어로 쓰면 포르투갈어로 그렁거리고, 스페인어·프랑스어도 마찬가지다. 번역하는 게 아니라 *스타일*만 압축한다. `wenyan` 모드만 의도적인 예외로, 고전 중국어가 토큰당 의미 밀도가 가장 높기 때문이다.

## 제공 기능

| 명령 | 하는 일 |
| --- | --- |
| `/caveman [lite\|full\|ultra\|wenyan]` | 모든 응답을 압축. 레벨은 세션 동안 유지 |
| `/caveman-commit` | Conventional Commit 메시지, 제목 50자 이하. what보다 why 중심 |
| `/caveman-review` | 한 줄짜리 PR 코멘트: `L42: 🔴 bug: user null. Add guard.` |
| `/caveman-stats` | 실제 세션 토큰 사용량, 누적 절감량, USD 환산. `--share`로 트윗용 한 줄 |
| `/caveman-compress <file>` | `CLAUDE.md` 같은 메모리 파일을 caveman 말투로 재작성. 이후 **매 세션마다** 입력 토큰 약 46% 절감. 코드·URL·경로는 바이트 보존 |
| `caveman-shrink` | MCP 미들웨어. 임의의 MCP 서버를 감싸 툴 설명을 압축 ([npm](https://www.npmjs.com/package/caveman-shrink)) |
| `cavecrew-*` | Caveman 서브에이전트(investigator, builder, reviewer). 기본 대비 약 60% 적은 토큰이라 메인 컨텍스트가 더 오래 간다 |

Claude Code에서는 상태줄에 `[CAVEMAN] ⛏ 12.4k`가 뜬다 — 누적 절감 토큰이며 `/caveman-stats`마다 갱신된다. `CAVEMAN_STATUSLINE_SAVINGS=0`으로 끌 수 있다.

## 벤치마크

Claude API의 실제 토큰 카운트 기준, 10개 프롬프트 평균 **출력 65% 감소**(범위 22~87%)를 기본 장황한 응답과 비교해 측정했다. 출력 토큰만 대상이며 `benchmarks/`와 `evals/`에 커밋되어 재현 가능하다.

| 작업 | Normal | Caveman | 절감 |
| --- | --- | --- | --- |
| React 리렌더 버그 설명 | 1180 | 159 | 87% |
| 인증 미들웨어 토큰 만료 수정 | 704 | 121 | 83% |
| PostgreSQL 커넥션 풀 설정 | 2347 | 380 | 84% |
| git rebase vs merge 설명 | 702 | 292 | 58% |
| 콜백을 async/await로 리팩터 | 387 | 301 | 22% |
| 아키텍처: 마이크로서비스 vs 모놀리스 | 446 | 310 | 30% |
| 보안 이슈 PR 리뷰 | 678 | 398 | 41% |
| Docker 멀티스테이지 빌드 | 1042 | 290 | 72% |
| PostgreSQL 레이스 컨디션 디버깅 | 1200 | 232 | 81% |
| React error boundary 구현 | 3454 | 456 | 87% |
| **평균** | **1214** | **294** | **65%** |

### 정직한 숫자 경고

Caveman은 **출력** 토큰만 줄인다. 입력·추론 토큰은 그대로이고, 스킬 자체가 턴당 약 1~1.5k 입력 토큰을 더한다. 따라서 세션 전체 절감은 출력 수치보다 작고, 이미 간결한 워크로드에서는 순손실이 될 수도 있다. 진짜 이득은 **가독성과 속도**이고 비용 절감은 보너스다. 언제 이기고 언제 지는지, 직접 측정하는 법은 `docs/HONEST-NUMBERS.md`에 있다.

짧은 게 싸기만 한 것도 아니다. 2026년 3월 논문 [*Brevity Constraints Reverse Performance Hierarchies in Language Models*](https://arxiv.org/abs/2604.00025)은 31개 모델을 테스트해, 큰 모델에 짧은 답변을 강제하면 일부 벤치마크에서 **정확도가 약 26포인트 향상**됐다고 보고했다. 때로는 말이 적을수록 더 정확하다.

### caveman-compress 실측

메모리 파일을 압축해 입력 토큰을 영구히 줄인 결과:

| 파일 | 원본 | 압축 후 | 절감 |
| --- | --- | --- | --- |
| `claude-md-preferences.md` | 706 | 285 | **59.6%** |
| `project-notes.md` | 1145 | 535 | **53.3%** |
| `claude-md-project.md` | 1122 | 636 | **43.3%** |
| `todo-list.md` | 627 | 388 | **38.1%** |
| `mixed-with-code.md` | 888 | 560 | **36.9%** |
| **평균** | **898** | **481** | **46%** |

이후 모든 세션에서 그 파일이 약 46% 작게 로드된다. 한 번이 아니라 계속 절약되는 입력 토큰이다.

## 동작 방식

1. 설치가 에이전트에 스킬 파일을 하나 떨어뜨린다.
2. 스킬이 에이전트에게 지시한다: filler 버리기, 알맹이 유지, 문장 조각 허용 — 단 코드·명령어·에러는 절대 건드리지 말 것.
3. Claude Code에서는 훅이 세션마다 작은 플래그 파일을 써서, `/caveman` 없이도 첫 메시지부터 caveman으로 말한다.
4. `/caveman-stats`가 세션 로그를 읽어 절감 토큰을 세고 그 숫자를 상태줄에 쓴다.
5. `/caveman-compress`가 `CLAUDE.md` 같은 메모리 파일을 재작성해, 이후 모든 세션이 더 작은 컨텍스트로 시작하게 한다.

훅 아키텍처, 파일 소유권, CI 동기화는 유지보수자용 `CLAUDE.md`에 문서화되어 있다.

## 생태계

이 스킬은 에이전트가 **말하는 것**을 줄인다. [caveman-code](https://github.com/JuliusBrussee/caveman-code)는 **전부**를 줄인다 — 위에서 아래까지 caveman인 풀 터미널 코딩 에이전트로, 동일 작업에서 **Codex 대비 토큰 약 2배 절감**. 20개 이상 프로바이더, plan 모드, autopilot goal loop, MIT.

```
npm install -g @juliusbrussee/caveman-code
```

다섯 개 도구, 하나의 아이디어: **에이전트가 더 적게 쓰고 더 많이 한다.**

| 저장소 | 무엇을 줄이나 |
| --- | --- |
| [**caveman**](https://github.com/JuliusBrussee/caveman) *(이 글)* | 에이전트가 **말하는 것** |
| [**caveman-code**](https://github.com/JuliusBrussee/caveman-code) | **에이전트 전체**, 처음부터 끝까지 |
| [**cavemem**](https://github.com/JuliusBrussee/cavemem) | 에이전트가 세션을 넘어 **기억하는 것** |
| [**cavekit**](https://github.com/JuliusBrussee/cavekit) | **빌드 루프** — 스펙 주도, 추측 없음 |
| [**cavegemma**](https://github.com/JuliusBrussee/finetune-caveman) | 가중치에 구워 넣은 압축 (Gemma 파인튜닝) |

한 번 설치로 따라오는 자매 스킬 다섯 개도 있다 — [JuliusBrussee/skills](https://github.com/JuliusBrussee/skills), Claude Code·Cursor·Gemini·Cline·Copilot 등 40개 이상 에이전트 지원:

| 스킬 | 내용 |
| --- | --- |
| **caveman** | 이 스킬. 적게 말하고 많이 전한다 |
| **grill-me** | 잘못된 걸 만들기 *전에* 에이전트가 계획을 취조한다 |
| **interface-kit** | 보기 좋고 빠르며 모두에게 동작하는 UI 만들기 |
| **junior-to-senior** | 적대적 리뷰 패스. 주니어 출력 in, 시니어 출력 out |
| **loop-factory** | 스펙 주도 태스크 루프 — inbox → active → archive |

```
npx skills@latest add JuliusBrussee/skills
```

### OpenClaw 연동

[OpenClaw](https://openclaw.ai/)는 한 박스에 여러 에이전트를 넣고 Slack/Discord/iMessage/Telegram에 연결하는 셀프호스트 게이트웨이다. 같은 인스톨러를 에이전트 하나로 좁혀 쓴다:

```
curl -fsSL https://raw.githubusercontent.com/JuliusBrussee/caveman/main/install.sh | bash -s -- --only openclaw
```

일어나는 일은 두 가지뿐이다. caveman 스킬이 워크스페이스에 설치되고, 마커로 감싼 작은 블록이 `SOUL.md`에 추가된다(OpenClaw가 매 턴 주입하므로 세션마다 `/caveman`을 칠 필요 없이 첫 메시지부터 간결하다). 경로를 바꾸려면 `OPENCLAW_WORKSPACE=/your/path`, 제거는 같은 명령에 `--uninstall`을 붙이면 되고 다른 워크스페이스 내용은 건드리지 않는다.

## Caveman 2와 프라이버시

현재의 절감 수치(`/caveman-stats` 포함)는 로컬 추정치다. Caveman 2는 이를 팀 단위로 측정·검증한다 — 실제 영수증, 실제 대시보드, 토큰이 정말 줄었다는 증명. 현재 개발 중이며 [caveman.so](https://caveman.so/)에서 대기자 명단을 받는다.

프라이버시: 텔레메트리·애널리틱스·계정·백엔드가 없다. 설치 후 네트워크 호출이 0이다 — 스킬은 프롬프트일 뿐이고, 훅은 로컬 스크립트이며, `/caveman-stats`는 이미 디스크에 있는 로그를 읽는다. 설치 시점의 fetch(GitHub 및 각 에이전트의 레지스트리)는 `SECURITY.md`에 명시되어 있다.

라이선스는 MIT. 스폰서로 [Atlas Cloud](https://www.atlascloud.ai/)가 있다.

## 관련 노트

- [headroom-context-compression](/posts/headroom-context-compression/) — LLM에 도달하기 전 입력 컨텍스트를 압축하는 반대편 접근
- [claude-code-model-and-effort](/posts/claude-code-model-and-effort/) — 출력·추론 토큰을 좌우하는 모델과 effort 선택
- [mattpocock-skills](/posts/mattpocock-skills/) — 실무자가 쓰는 Claude Code 스킬 모음
- [midudev-autoskills](/posts/midudev-autoskills/) — 스킬 스택을 명령 하나로 설치하는 도구
- [alirezarezvani-claude-skills](/posts/alirezarezvani-claude-skills/) — 여러 코딩 에이전트를 아우르는 대규모 스킬 컬렉션

> 원문: [JuliusBrussee/caveman: 🪨 why use many token when few token do trick — Claude Code skill that cuts 65% of tokens by talking like caveman](https://github.com/juliusbrussee/caveman)
> 원본 클립: 2026-07-26-JuliusBrusseecaveman 🪨 why use many token when few token do trick — Claude Code skill that cuts 65% of tokens by talking like caveman
