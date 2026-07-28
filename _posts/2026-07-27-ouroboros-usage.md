---
title: '우로보로스 사용법'
date: 2026-07-27 11:18:00 +0900
categories: [기록, 도구]
tags: [ouroboros, claude-code, plugin, mcp, spec-first, workflow]
description: 'Ouroboros 플러그인을 실제로 쓰면서 정리한 명령·플래그·운영 요령. 무엇인지에 대한 설명은 정리본 노트에 있고, 여기는 내 환경에서 어떻게 굴리는가만 다룬다.'
---
[Ouroboros](/posts/ouroboros-agent-os/) 플러그인을 실제로 쓰면서 정리한 명령·플래그·운영 요령. 무엇인지에 대한 설명은 정리본 노트에 있고, 여기는 **내 환경에서 어떻게 굴리는가**만 다룬다.

버전: 0.50.5 (플러그인 `ouroboros@ouroboros`)
설치 위치: `~/.claude/plugins/marketplaces/ouroboros/`
상태 저장소: `~/.ouroboros/` (SQLite `ouroboros.db`, `seeds/`, `logs/`, `locks/`)

---

## 1. 우로보로스가 뭔가

**요구사항 [결정화](/posts/crystallization/)(crystallization) 엔진.** 애매한 아이디어를 즉흥 프롬프트로 때우지 말고, 구조화된 워크플로로 검증된 코드까지 끌고 간다.

핵심 5요소:

1. **[소크라테스 인터뷰](/posts/socratic-interview/)** — 숨은 가정을 질문으로 드러냄
2. **[Seed](/posts/seed-spec/) 생성** — 불변 YAML 스펙 생성
3. **[PAL 라우팅](/posts/pal-routing/)** — 작업 복잡도에 따라 모델 자동 승급/강등
4. **[측면 사고](/posts/lateral-thinking/)(lateral thinking)** — 5개 페르소나로 정체 돌파
5. **3단 평가** — 기계적 → 의미적 → 합의

전체 흐름:

```
interview  →  seed  →  run  →  evaluate  →  evolve
(질문으로 요구사항 파기)  (불변 스펙)  (실행)  (검증)  (재진입/개선)
```

이 루프는 선형 단계가 아니라 **진화 루프**다. `evaluate`에서 실패하면 `evolve`로 스펙 자체를 다시 깎고 재실행한다.

---

## 2. 핵심 개념

| 개념 | 설명 |
|---|---|
| **Seed Spec** | 인터뷰에서 자동 생성되는 **불변** YAML. 프로젝트 설명 + AC 트리 + 제약 + 아키텍처 결정. "무엇을 만들지"의 유일한 진실. `~/.ouroboros/seeds/seed_<id>.yaml` |
| **[AC 트리](/posts/ac-tree/)** | Acceptance Criteria 계층 분해. 각 AC 상태: `pending` / `in_progress` / `passed` / `failed`. 진행률 추적 단위 |
| **이벤트 소싱** | 모든 상태 변경이 불변 이벤트로 SQLite에 적재. 과거 상태 완전 재구성 가능 |
| **드리프트 감지** | 실행이 원래 Seed 의도에서 벗어나는지 감시 |
| **Lineage(계보)** | 이벤트·결정의 인과 사슬. `evolve`/`ralph`가 이 ID로 세대를 관리 |
| **3단 평가** | 1단 기계적(무료: lint/build/test/커버리지) → 2단 의미적(AC 준수·목표 정렬·드리프트) → 3단 다중모델 합의(불확실할 때만) |

> Seed는 직접 손으로 안 쓴다. 인터뷰가 만든다. 수동 작성은 고급 워크플로(`docs/guides/seed-authoring.md`).

---

## 3. 명령 표기 두 가지

문서에는 `ooo <cmd>` 로 적혀 있고, Claude Code 세션에서 실제 입력은 `/ouroboros:<cmd>`. 둘은 같은 것.

```
ooo auto "..."          ==  /ouroboros:auto "..."
```

자연어 트리거도 붙어 있다 — "나 막혔어"라고 하면 `unstuck`, "PRD 써줘"면 `pm`이 걸린다.

표준 파이썬 CLI(`ouroboros ...`)는 별개 경로다. 이 머신에는 **미설치**. 필요하면:

```bash
pip install "ouroboros-ai[mcp,claude]"     # Python >= 3.12
```

---

## 4. 최단 경로 (추천)

```
/ouroboros:setup                          # 1회. MCP 서버 등록 + 프로젝트 설정 (~1분)
/ouroboros:auto "할 일 관리 CLI 만들어"      # 인터뷰~Seed~품질게이트~실행 자동
```

`auto`가 하는 일:

- 경계 있는 인터뷰 라운드 진행
- A등급 Seed 생성. B/C등급이면 자체 수리 시도
- **A등급 게이트 통과 후에만** 실행 시작
- `job_id` + `auto_session_id` 반환 (둘 다 보관)

`auto`는 오래 걸려서 대화형 MCP 타임아웃을 넘긴다. 그래서 백그라운드로 던지고 ID만 즉시 돌려준다. 끊기면:

```
/ouroboros:auto --resume auto_abc123
```

### auto 플래그

| 플래그 | 뜻 |
|---|---|
| `--skip-run` | Seed까지만 만들고 실행 안 함 |
| `--complete-product` | 실행 뒤 Ralph 루프까지 물려서 완제품까지 (기본 10세대) |
| `--resume <id>` | 중단된 auto 세션 재개 |
| `--max-interview-rounds N` | 인터뷰 라운드 상한 |
| `--max-repair-rounds N` | Seed 수리 라운드 상한 |
| `--pipeline-timeout-seconds X` | 파이프라인 전체 데드라인. **시작할 때만** 유효 (resume에는 못 붙임) |
| `--efficiency-mode adaptive\|quality_first` | 효율 우선 / 품질 우선 |
| `--frugality-assurance off\|observe\|strict` | 절약 검증 강도. `strict`는 별도 명시 opt-in |

조합 관례: **효율 실행** = `adaptive` + `observe`, **품질 우선 실행** = `quality_first` + `off`.

주의: `--max-generations`는 `auto` 플래그 아님 — `ralph` 것이다.
주의: `auto`는 부모 Seed를 안 받는다. 기존 Seed 파생은 `evolve`, 기존 Seed 실행은 `run`.

---

## 5. 수동 경로 (질문 직접 답하고 싶을 때)

```
/ouroboros:interview "할 일 관리 CLI 만들어"    # 소크라테스 인터뷰 → Seed 자동 생성
/ouroboros:run <seed_파일경로>                 # AC를 태스크로 분해해 실행
/ouroboros:evaluate <session_id>              # 3단 검증
/ouroboros:evolve "..."                       # 결과 반영해 스펙 진화 + 재실행
```

- `/ouroboros:seed [session_id]` — 보통 직접 안 부른다. 인터뷰가 자동 호출. 인터뷰 결과에서 Seed만 다시 뽑고 싶을 때 사용
- `/ouroboros:run` 인자는 Seed 파일 경로 또는 Seed 내용
- `/ouroboros:evaluate <session_id> [artifact]`

---

## 6. 진화 루프 두 가지

### evolve — 스펙 수렴용

```
ooo evolve "build a task management CLI"                  # 새 진화 루프 시작
ooo evolve "build a task management CLI" --no-execute      # 온톨로지만, 실행 없음 (빠름)
ooo evolve --status <lineage_id>                           # 계보 상태 확인
ooo evolve --rewind <lineage_id> <generation_number>       # 이전 세대로 되감기
```

스펙을 반복해서 깎아 수렴시킬 때. 되감기가 되는 게 강점.

### ralph — "끝날 때까지 안 멈춤"

```
ooo ralph --lineage-id <lineage_id>
```

백그라운드 `evolve_step` 잡을 클라이언트가 계속 돌린다. 검증 통과할 때까지 반복. 트리거 어구: "don't stop", "until it works", "keep going".

맨 자연어 요청만 있으면 `interview` + `seed` 먼저 하고, 새 `lineage_id`와 검증된 Seed YAML로 호출.

**선택 기준:** 스펙이 흔들려서 수렴이 필요하면 `evolve`. 스펙은 확정됐고 통과까지 갈아야 하면 `ralph`.

---

## 7. 막혔을 때 — unstuck

```
ooo lateral                       # 기본: 5개 페르소나 전원 토론
ooo lateral <persona>             # 단독 페르소나
ooo lateral debate <p1> <p2> ...  # 지정 멤버 토론
ooo lateral @all                  # 프리셋 (현재 @all만)
```

| 페르소나 | 태도 | 쓸 때 |
|---|---|---|
| **hacker** | 일단 돌게 만들어, 우아함은 나중 | 과한 고민이 진행을 막을 때 |
| **researcher** | 우리가 모르는 정보가 뭐냐 | 문제가 불명확할 때 |
| **simplifier** | 스코프 잘라, MVP로 돌아가 | 복잡도가 감당 안 될 때 |
| **architect** | 접근 자체를 재구성 | 현재 설계가 틀렸을 때 |
| **contrarian** | 우리가 틀린 문제를 풀고 있는 거 아니냐 | 가정을 흔들어야 할 때 |

이 다섯은 **상태 없는 사고방식 렌즈**다([측면 사고](/posts/lateral-thinking/)). evaluator·ontologist 같은 상태 있는 역할은 이 풀에 안 섞이고 각자 SKILL이 있다.

---

## 8. 나머지 명령

| 명령 | 용도 |
|---|---|
| `/ouroboros:status [session_id]` | 세션 상태 + 원래 목표 대비 드리프트 측정 |
| `/ouroboros:cancel` | 인터랙티브: 활성 실행 목록에서 골라 취소 |
| `/ouroboros:cancel <execution_id>` | 특정 실행 취소 |
| `/ouroboros:cancel --all` | 전체 실행 취소 |
| `/ouroboros:resume-session` | 진행 중 세션 목록 + 재접속 명령. MCP 끊겼을 때 |
| `/ouroboros:resume-session --all` | 전부 표시 |
| `/ouroboros:brownfield` | 레포 스캔 + 인터뷰 기본 레포 지정 |
| `/ouroboros:brownfield scan` | 스캔만 |
| `/ouroboros:brownfield defaults` | 현재 기본값 표시 |
| `/ouroboros:brownfield set 6,18,19` | 번호로 기본 레포 지정 |
| `/ouroboros:brownfield detect [path]` | AI 1회 호출로 `mechanical.toml` 작성 |
| `/ouroboros:pm` | PM 인터뷰 → PRD 생성 |
| `/ouroboros:qa [파일경로\|텍스트]` | 임의 산출물 QA 판정. 인자 없으면 최근 실행 결과 평가 |
| `/ouroboros:publish [seed_path]` | Seed를 GitHub Issue로 발행 (팀 작업용) |
| `/ouroboros:config` | 설정 GUI. 단계별 에이전트·모델 지정 (브라우저 / 터미널이면 TUI) |
| `/ouroboros:update` | 업데이트 확인 + 최신으로 업그레이드 |
| `/ouroboros:tutorial` | 실습형 인터랙티브 튜토리얼 |
| `/ouroboros:help` | 전체 레퍼런스 |
| `/ouroboros:welcome` | 첫 사용 안내 |

---

## 9. 에이전트

| 에이전트 | 역할 |
|---|---|
| `ouroboros:socratic-interviewer` | 질문으로 숨은 가정 노출 ([소크라테스 인터뷰](/posts/socratic-interview/)) |
| `ouroboros:ontologist` | 증상 말고 근본 문제 찾기 |
| `ouroboros:seed-architect` | 요구사항을 [Seed 스펙](/posts/seed-spec/)으로 [결정화](/posts/crystallization/) |
| `ouroboros:evaluator` | 3단 검증 |
| `ouroboros:contrarian` / `hacker` / `simplifier` / `researcher` / `architect` | [측면 사고](/posts/lateral-thinking/) 5인 |

---

## 10. Plugin 모드 vs MCP 모드

- **Plugin** — `setup` 직후 바로 동작: `interview`, `seed`, `unstuck`, `setup`, `welcome`, `tutorial`, `help`, `qa`, `update`, `publish`, `ralph`(일부)
- **MCP** — Python >= 3.12 자동 감지 필요, `setup` 1회로 해금: `run`, `evaluate`, `status`, `evolve`, `pm`, `brownfield`, `auto`
- **CLI** — `config`, `cancel`, `resume-session`

MCP 도구는 `mcp__plugin_ouroboros_ouroboros__ouroboros_*` 이름으로 노출된다. 스키마가 지연 로딩(deferred)이라 도구 목록에 안 보일 수 있음 — 없는 게 아니다.

---

## 11. 런타임 백엔드

`orchestrator.runtime_backend`로 설정.

- **Claude Code** — 주 권장 백엔드. MCP 도구 연동
- **Codex CLI** — OpenAI Codex CLI 대안
- provider 시스템으로 추가 백엔드 연결 가능

---

## 12. 문서 위치

`~/.claude/plugins/marketplaces/ouroboros/docs/` 아래:

| 파일 | 내용 |
|---|---|
| `getting-started.md` | 온보딩 단일 진실 |
| `architecture.md` | 시스템 설계 |
| `config-reference.md` | 설정 키·기본값 ([PAL 라우팅](/posts/pal-routing/) 티어 정의 포함) |
| `runtime-capability-matrix.md` | 백엔드별 기능 비교 |
| `guides/seed-authoring.md` | 수동 Seed 작성 (고급) |
| `runtime-guides/claude-code.md`, `runtime-guides/codex.md` | 백엔드별 설정 |
| `README.ko.md` (루트) | 한국어 README (정리본) |
| `llms.txt`, `llms-full.txt` (루트) | LLM용 압축 레퍼런스 |

---

## 13. 자주 쓰는 시나리오

**신규 프로젝트, 빠르게:**
```
/ouroboros:auto "아이디어" --efficiency-mode adaptive
```

**신규 프로젝트, 품질 우선 완제품까지:**
```
/ouroboros:auto "아이디어" --complete-product --efficiency-mode quality_first
```

**스펙만 먼저 확정하고 실행은 검토 후:**
```
/ouroboros:auto "아이디어" --skip-run
# Seed 확인 후
/ouroboros:run ~/.ouroboros/seeds/seed_<id>.yaml
```

**기존 레포에 붙이기:**
```
/ouroboros:brownfield          # 레포 스캔 + 기본값 지정
/ouroboros:interview "붙일 기능"
/ouroboros:run
```

**실행이 이상하게 흘러갈 때:**
```
/ouroboros:status              # 드리프트 확인
/ouroboros:unstuck             # 5인 토론으로 재구성
```

**멈춘 실행 정리:**
```
/ouroboros:resume-session      # 살아있는지 확인
/ouroboros:cancel --all        # 아니면 전부 취소
```

---

## 관련

- [결정화](/posts/crystallization/) · [소크라테스 인터뷰](/posts/socratic-interview/) · [Seed 스펙](/posts/seed-spec/) · [AC 트리](/posts/ac-tree/) · [PAL 라우팅](/posts/pal-routing/) · [측면 사고](/posts/lateral-thinking/) — 핵심 개념 용어 노트
- [Q00/ouroboros — 프롬프트를 멈추고 명세를 시작하는 Agent OS](/posts/ouroboros-agent-os/) — 영문 README 정리본
- [Ouroboros 한국어 README — 명세 우선 AI 개발 시스템](/posts/ouroboros-readme-ko/) — 한국어판 고유 내용
- [Caveman 사용법](/posts/caveman-usage/) — 같은 성격의 플러그인 운영 메모
- [Loop Engineering](/posts/loop-engineering/) — 이 워크플로가 속한 더 큰 그림
