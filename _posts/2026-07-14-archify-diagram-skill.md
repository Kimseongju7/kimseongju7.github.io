---
title: 'Archify — 채팅으로 아키텍처 다이어그램을 그려주는 에이전트 스킬 (다크/라이트 토글 + PNG/SVG 내보내기)'
date: 2026-07-14 14:25:41 +0900
categories: [개발, 도구]
tags: [skill, diagram, architecture, claude-code, codex, svg, ai-agent]
description: '채팅으로 시스템·프로세스를 설명하면 다크/라이트 토글과 PNG/SVG 내보내기가 되는 아키텍처 다이어그램을 만들어주는 에이전트 스킬 Archify 정리.'
---
Archify(tt-a1i/archify)는 Claude, Codex CLI, opencode용 에이전트 스킬로, 시스템이나 프로세스를 평이한 영어로 설명하면 다듬어진 자기완결형(self-contained) HTML 다이어그램 파일로 만들어 준다. 브라우저에서 열어 다크/라이트 테마를 토글하고, 클립보드로 복사하거나 최대 4배 해상도의 PNG/JPEG/WebP/SVG로 내보낼 수 있다.

## 핵심 특징

- **디자인 스킬 불필요** — 아키텍처를 영어로 설명하면 다이어그램이 나온다
- **다이어그램 5종** — 아키텍처 외에 워크플로, 시퀀스, 데이터 플로, 라이프사이클(승인 흐름, 도구 호출, CI/CD, 런북, 요청 콜체인, 데이터 파이프라인, PII 경계, 상태 머신 등)
- **내장 테마 토글** — 클릭 한 번으로 다크/라이트 전환, 세션 간 유지
- **PNG 클립보드 복사** — 한 번의 클릭으로 Slack / Notion / GitHub에 바로 붙여넣기
- **초고선명 내보내기** — PNG/JPEG/WebP를 원본 해상도의 최대 4배로 브라우저가 네이티브 렌더링(업샘플링 블러 없음), 진짜 벡터가 필요하면 SVG
- **SVG가 시스템 다크/라이트를 따름** — 내보낸 SVG에 두 테마의 변수 세트와 `@media (prefers-color-scheme)`이 함께 담겨, GitHub README에 하나만 넣어도 읽는 사람의 색상 설정을 따른다(`<picture>`로 PNG 두 장 감싸던 시절 끝)
- **검증 루프 내장** — 렌더러 기반 다이어그램은 JSON 스키마 검증, 레이아웃 체크, HTML/SVG 아티팩트 체크, 타깃 반복(iteration)을 거친다
- **시맨틱 기술 라벨** — 컴포넌트를 `aws.lambda`, `postgres`, `redis`, `github-actions`, `openai`처럼 기술하면 전체 아이콘 라이브러리 없이 알맞은 시각 카테고리로 매핑
- **의존성 제로의 단일 HTML** — 파일 하나를 보내는 것으로 공유 완료
- **채팅으로 반복 수정** — "Redis 추가해줘", "auth를 왼쪽으로", "API는 에메랄드로"

## 다이어그램 유형 5가지

| 유형 | 적합한 대상 | 요청 방법 |
| --- | --- | --- |
| Architecture | 시스템 컴포넌트, 클라우드 리소스, DB, 캐시, 서비스, 경계, 보안 그룹 | 시스템 구조를 설명 |
| Workflow | 요청 라이프사이클, 승인 흐름, 도구 호출, CI/CD, 런북, 장애 대응 | 참여자·단계 순서·주요 분기 설명 |
| Sequence | API 콜체인, 캐시 폴백, 인증 검사, 비동기 트레이스, 서비스 상호작용 | 누가 누구를 어떤 순서로 부르고 무엇이 반환되는지 설명 |
| Data Flow | 데이터 파이프라인, ETL/ELT, 분석 이벤트, PII 격리, 웨어하우스 동기화, 리니지 | 소스·처리 단계·저장·민감도 경계·소비자 설명 |
| Lifecycle | 상태 머신, 주문/작업/배포/에이전트 실행 라이프사이클, 대기·재시도·취소·타임아웃·종결 상태 | 상태·전이 이벤트·재시도 경로·종결 결과 설명 |

- Workflow는 범용 순서도를 대체하려는 것이 아니라 기술 커뮤니케이션용 다이어그램이다: 스윔레인, 시맨틱 컬러, 명확한 해피 패스, 보조적인 비동기/승인/트레이스 경로.
- 예: `Use archify to draw a workflow: User submits a request -> Agent plans -> Approval Gate if needed -> Tool Call -> Trace Log -> Final Reply`

## 빠른 시작

```
npx skills add tt-a1i/archify -g
```

설치 후 에이전트에게 `Use archify to map this repository's runtime architecture.`라고 요청하면 된다. 영구 설치 없이 써보려면 `npx skills use tt-a1i/archify@archify --agent codex` (codex 자리에 `claude-code`나 `opencode` 가능). 이 명령들은 오픈소스 `skills` CLI(vercel-labs/skills)를 쓴다.

- 배포판 스킬에는 독립 실행형 스키마 검증기가 포함돼 있어 `npm install`이나 런타임 의존성 없이 바로 렌더링할 수 있다.
- 다른 설치 방법: `archify.zip`을 Claude.ai(Settings → Capabilities → Skills 업로드), Claude Code CLI(`~/.claude/skills/`에 unzip), Codex CLI(`~/.agents/skills/`), opencode(`~/.config/opencode/skills/`)에 설치.
- 설치 표면별 기능: Claude Code / Codex CLI / opencode는 완전 기능(타입 렌더러 + 스키마 검증), Claude.ai zip 업로드는 샌드박스의 Node.js 실행 허용 여부에 따라 대체로 완전, Project Knowledge는 코드 실행이 없어 아키텍처 모드만.

## 품질 루프 (렌더러 기반 다이어그램)

| 단계 | 하는 일 |
| --- | --- |
| JSON IR 생성 | 최종 SVG를 손으로 고치는 대신 타입이 있는 다이어그램 기술(description)을 작성 |
| 검증 | 번들된 독립형 검증기가 JSON 스키마 체크 |
| 렌더 | 타입 렌더러가 자기완결형 HTML/SVG 출력 생성 |
| 아티팩트 체크 | 잘못된 SVG, 비유한 좌표, 우발적 대각선 화살표, 범례를 가로지르는 경로를 포착 |
| 반복 | 수정은 JSON IR이나 시맨틱 클래스에 대해 이루어져, 전체 재생성 없이 국소 수정 가능 |

- 최종 체크 직접 실행: `node scripts/check-render-output.mjs output.html`
- 번들 CLI: `node bin/archify.mjs`의 `doctor` / `demo` / `render` / `validate` / `check` / `examples` / `inspect` 명령
- 데모용 모션: `{ "meta": { "animation": "trace" } }`로 옵트인 트레이스 애니메이션(화살표가 순서대로 흐르고 노드가 가볍게 펄스). `prefers-reduced-motion` 사용자에게는 자동 비활성화.

## 출력 사용법

- 생성된 HTML을 브라우저에서 열면 우상단에 테마 버튼(단축키 T)과 Export 메뉴(단축키 E)가 있다.
- Export 메뉴: Copy PNG(클립보드로 바로), PNG/JPEG/WebP 다운로드(JPEG/WebP는 현재 테마 배경으로 채움, PNG는 투명 유지), SVG 다운로드(스타일 인라인, 듀얼 테마 자기완결형, Figma/Illustrator에서 편집 가능).
- 모든 래스터 내보내기는 브라우저가 다이어그램 고유 해상도의 최대 4배로 네이티브 래스터화한다(캔버스 한계를 넘으면 자동으로 3×/2×로 다운스텝). 스케일 다이얼은 없고 항상 안전한 최대 선명도를 자동 선택.
- 키보드: T(테마), E(Export), 메뉴 안 ↑↓/Home/End/Enter/Esc.
- URL 파라미터: `?theme=light|dark`(시작 테마 고정), `?openExport=1`(로드 시 Export 메뉴 자동 열기) — 결정적 스크린샷용.
- 참고: WebP 내보내기는 브라우저 캔버스 인코더에 의존(구형 Safari 미지원 시 비활성), 이미지 클립보드는 `ClipboardItem` + `navigator.clipboard.write` 필요(Chromium, Firefox 127+, Safari 16+), 래스터 내보내기 폰트는 시스템 모노스페이스 폴백(픽셀 퍼펙트를 원하면 JetBrains Mono 로컬 설치).

## 색상 팔레트와 시맨틱 라벨

| 컴포넌트 유형 | 색 | 용도 |
| --- | --- | --- |
| Frontend | 시안(Cyan) | 클라이언트 앱, UI, 엣지 디바이스 |
| Backend | 에메랄드(Emerald) | 서버, API, 서비스 |
| Database | 바이올렛(Violet) | DB, 스토리지, AI/ML |
| Cloud / AWS | 앰버(Amber) | 클라우드 서비스, 인프라 |
| Security | 로즈(Rose) | 인증, 보안 그룹, 암호화 |
| Message Bus | 오렌지(Orange) | Kafka, RabbitMQ, SNS, 이벤트 버스 |
| External | 슬레이트(Slate) | 범용, 외부 시스템 |

각 색은 다크/라이트 모드 변형이 짝을 이뤄 테마 토글 시 함께 전환된다. Archify는 AWS/Azure/GCP 아이콘 런타임을 통째로 싣는 대신 기술 이름을 시맨틱 힌트로 취급한다: `react`/`nextjs`/`browser`→Frontend, `node`/`api-gateway`→Backend, `postgres`/`redis`/`s3`→Database/storage, `aws.lambda`/`kubernetes`→Cloud, `auth0`/`vault`→Security, `kafka`/`sqs`→Message bus, `stripe`/`openai`/`slack`→External 등.

## 기술 세부와 히스토리

- 스타일링: `:root` + `[data-theme="light"]`의 CSS 커스텀 프로퍼티, SVG 요소는 시맨틱 클래스(`c-frontend`, `t-muted`, `a-emphasis` 등) 참조. `<html>`의 `data-theme` 토글만으로 그라디언트·그리드·화살표·마스크까지 전체 리테마.
- 내보내기 파이프라인: SVG를 복제해 스타일 인라인 → 현재 테마 변수를 해석해 기록 → 래스터는 `4 × viewBox` 크기로 브라우저 네이티브 래스터화(비트맵 업샘플링 없음) → `toBlob(mime)`으로 파일 생성. 캔버스 한계 초과 시 3×/2×로 자동 폴백.
- 출력물: 단일 HTML(구글 폰트 링크 + 인라인 SVG + 약 19KB 임베디드 JS). 빌드 스텝, JS 프레임워크, 서버 없음.
- Archify는 Cocoon-AI/architecture-diagram-generator v1.0(MIT, 다크 전용)의 포크/재작성이다. 2.x에서 테마 시스템, 내보내기 파이프라인, 클립보드 복사, 4× 네이티브 래스터화, 듀얼 테마 SVG, 접근성, 렌더러 기반 5종 다이어그램 + 스키마 검증, 사후 아티팩트 체크, 통합 CLI 등을 추가했다(2.0~2.10). 둘 다 MIT 라이선스.
- 로드맵: 다음은 v3.0 — JSON IR 안정화. 최소한의 `diagram.json` 중간 포맷으로 Claude가 무관한 컴포넌트를 흔들지 않고 국소 좌표 수정을 할 수 있게 하고, `git diff` 친화적 출력과 재렌더링 없는 테마/팔레트 교체를 목표로 한다. Mermaid 자동 파서 경로는 실험 결과(자동 레이아웃 + archify CSS가 네이티브 Mermaid보다 의미 있게 낫지 않음) 폐기됐다 — Archify의 미적 핵심은 CSS가 아니라 Claude의 레이아웃 판단이며, Mermaid 코드를 붙여넣으면 파서가 아니라 SKILL.md 프롬프트를 통해 처음부터 레이아웃한다.

## 관련 노트

- [mattpocock-skills](/posts/mattpocock-skills/) — 작고 조합 가능한 엔지니어링 스킬 모음
- [midudev-autoskills](/posts/midudev-autoskills/) — 스택을 감지해 스킬을 자동 설치하는 도구
- [codex-top-6-skills](/posts/codex-top-6-skills/) — Codex 에이전트를 확장하는 대표 스킬 6선

> 원문: [tt-a1i/archify: Any agent Skill: generate beautiful architecture diagrams with dark/light theme toggle and PNG/JPEG/WebP/SVG export](https://github.com/tt-a1i/archify)
> 원본 클립: 2026-07-14-tt-a1iarchify Any agent Skill generate beautiful architecture diagrams with darklight theme toggle and PNGJPEGWebPSVG export
