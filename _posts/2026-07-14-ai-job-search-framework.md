---
title: 'MadsLorentzen/ai-job-search — Claude Code 기반 AI 구직 프레임워크'
date: 2026-07-14 14:18:23 +0900
categories: [개발, 도구]
tags: [claude-code, job-search, cv, latex, agent, workflow, career]
description: 'Claude Code를 구직 도우미로 바꿔 채용 공고 평가, CV·커버레터 작성, 면접 준비까지 자동화하는 오픈소스 프레임워크 ai-job-search 정리.'
---
**ai-job-search**는 Claude Code를 풀스택 구직 도우미로 바꿔주는 오픈소스 프레임워크다. 포크해서 프로필을 채우면 Claude가 채용 공고를 평가하고, CV를 맞춤 수정하고, 커버레터를 쓰고, 면접 준비까지 해준다. 만든 이가 자신의 구직에 직접 써서 69건의 맞춤 지원, 20번의 1차 면접, 1건의 계약 서명으로 2026년 6월 AI 엔지니어로 입사한 실전 검증 사례가 붙어 있다.

## 실제로 효과가 있었나

저자는 지구물리학자 출신으로 2025년 말 자리가 없어지자 이 프레임워크를 만들어 매주 자신의 구직에 사용했다. 모든 고용주에게 이 도구를 쓴다는 사실을 솔직히 밝혔는데, 불이익이 되기는커녕 보통 진지한 기술 대화로 이어졌다고 한다. 참고로 이 프로젝트는 Anthropic과 무관한 독립 오픈소스이며, 연관 암호화폐·토큰·유료 스폰서십 프로그램이 전혀 없다(그런 주장은 전부 사기).

## 무엇인가

핵심 워크플로(자기 프로파일링, 적합도 평가, 드래프터-리뷰어 지원서 파이프라인)는 **언어·국가 불문**으로 동작한다. 채용 포털 검색 스킬은 덴마크 시장(Jobindex, Jobnet, Akademikernes Jobbank 등)용으로 만들어져 있지만, 자기 지역 채용 사이트로 교체할 수 있는 패턴으로 설계됐다.

```
/setup → 프로필 작성 → 프로필 파일 준비
/scrape → 포털 검색 → 적합도와 함께 매칭 제시 → 선택 → /apply
/apply <url> → 적합도 평가 → CV+커버레터 초안(LaTeX) → 리뷰어 에이전트 비평 → 수정 → 최종 출력
```

## 사전 준비물

- Claude Code(CLI), Python 3.10+, Bun(채용 검색 CLI 도구용)
- `lualatex`와 `xelatex`를 포함한 LaTeX 배포판(TeX Live, MacTeX, TinyTeX, MiKTeX). CV는 `lualatex`로 컴파일(pdflatex는 최신 MiKTeX에서 `fontawesome5` 오류로 자주 실패), 커버레터는 `cover.cls`가 `fontspec`을 요구해 `xelatex`로 컴파일한다.
- 선택: poppler의 `pdftotext` — `/apply`의 ATS 파싱 가능성 검사에 사용. 없으면 시각적 키워드 리뷰로 우아하게 강등된다.

## 빠른 시작

1. `gh repo fork MadsLorentzen/ai-job-search --clone`로 포크·클론
2. `.agents/skills/*/cli`에서 각 채용 검색 도구를 `bun install` (linkedin-search와 freehire-search는 런타임 의존성 0으로 설치가 선택 사항)
3. Claude Code 안에서 `/setup` — documents/ 폴더 읽기, 채팅에 CV 붙여넣기, 인터뷰 진행의 세 경로를 제공하며 자동 감지해 물어본다. documents 폴더 모드는 멱등이라 자료를 추가하며 재실행해도 안전하다.
4. `/scrape` — 여러 포털을 검색해 중복 제거 후 적합도 순으로 제시. 결과가 많으면 `/rank`로 일괄 점수화해 랭킹 숏리스트를 먼저 받을 수 있다.
5. `/apply <URL>` — URL을 못 가져오는 포털이면 공고 본문을 직접 붙여넣어도 된다.

## /apply의 동작 방식

`/apply`는 PDF 컴파일이 필수인 **드래프터-리뷰어 워크플로**를 돌린다: 공고 파싱 → 프로필 대비 적합도 평가 → LaTeX로 CV·커버레터 초안 → 회사를 조사하고 초안을 비평하는 리뷰어 에이전트 스폰 → 피드백 반영 수정 → 두 PDF 컴파일·검사(CV는 정확히 2쪽, 커버레터는 정확히 1쪽에 서명 표시·폰트 일관까지 반복 수정) → ATS 검사 → 검증 체크리스트와 함께 최종 제시. CV와 커버레터의 모든 주장은 실제 프로필과 대조 검증되며, 스킬이나 경력을 절대 지어내지 않는다.

이 워크플로가 다른 것들과 차별되는 지점:

- **PDF 검증 루프** — .tex에서는 멀쩡해 보여도 PDF에서 깨지는 문제(제목이 다음 쪽으로 고아가 되거나, 커버레터가 2쪽으로 넘치거나, 폰트가 조용히 폴백)를 컴파일 후 시각 검사로 잡고 `\needspace`, `\enlargethispage` 같은 표적 수정을 자동 적용한다.
- **PDF 텍스트 레이어 ATS 검증** — ATS는 렌더링된 페이지가 아니라 PDF에 내장된 텍스트를 읽는다. `pdftotext`로 텍스트 레이어를 추출해 연락처가 리터럴 텍스트로 존재하는지, 글리프 깨짐이 없는지, 읽기 순서가 정상인지, 공고 키워드 커버리지를 검증한다. 프로필이 뒷받침하지 못하는 키워드는 갭으로 인정하지 절대 끼워넣지 않는다(정직성 규칙).
- **관련도 가중 CV 컷** — CV가 2쪽을 넘치면 오래된 섹션부터 기계적으로 자르지 않는다. 각 줄을 (a) 대상 공고와의 관련성, (b) 문서 내 고유성, (c) 커버레터 의존 여부로 점수화해 합계가 낮은 줄부터 자른다.
- **드래프터-리뷰어 분리** — 새 컨텍스트로 스폰된 두 번째 Claude 에이전트가 회사를 조사하고 초안을 비평한다. 한 번에 쓰면 남기 쉬운 놓친 키워드, 약한 프레이밍, 뻔한 표현을 잡아낸다.
- **토큰 효율적 리뷰어 디스패치** — 리뷰어가 초안을 다시 읽지 않고 인라인으로 받으며, 검증 체크리스트는 워크플로 끝에 한 번만 돈다.

## 확장 명령어

핵심 3개(`/setup`, `/scrape`, `/apply`) 외 7개가 더 있다.

- **`/interview`** — 추적 중인 지원 건의 면접을 위해 단계별 준비 팩을 만든다. 실제 제출한 공고·CV·커버레터·이전 라운드 피드백을 아카이브에서 꺼내고, 회사·면접관을 검증-후-사용 규칙으로 조사하고, 예상 질문을 STAR 예시에 매핑하며, 모의 면접도 제공한다. 갭에는 정직한 브릿지 답변을 주지 경험을 지어내지 않는다.
- **`/outcome`** — 지원 결과(면접 단계, 오퍼, 탈락, 무응답)를 기록하고 제출물을 `documents/applications/<company>_<role>/`에 아카이브하며 트래커를 갱신한다. 몇 건이 결론 나면 실제로 면접을 딴 데이터로 적합도 프레임워크를 보정하라고 안내한다.
- **`/rank`** — 새로 긁은 공고들을 병렬 에이전트로 5개 평가 차원 일괄 채점해 랭킹 숏리스트를 반환. 딜브레이커는 거부권, 마감은 긴급 플래그, 죽은 공고는 만료 처리.
- **`/expand`** — 프로필에 이미 링크된 공개 소스(GitHub, 포트폴리오, Kaggle, Google Scholar)와 수료 과정 실라버스를 스캔해 역량을 출처 태그와 함께 프로필에 추가.
- **`/upskill`** — 프로필과 추적 중인 공고들(또는 단일 공고) 사이 스킬 갭을 분석해 우선순위 히트맵과 학습 계획(웹 검색한 학습 자료 + 시간 추정)을 산출.
- **`/add-template`** — 자기 LaTeX CV/커버레터 템플릿을 등록. 컴파일 엔진·폰트·스타일 규칙·페이지 제한을 인터뷰로 수집하고 필수 테스트 컴파일을 돌린 뒤 `/apply`에 연결. 템플릿은 개인정보 대신 `[PLACEHOLDER]` 토큰으로 저장돼 커밋·공유에 안전하다.
- **`/add-portal`** — 자기 시장의 채용 사이트용 검색 스킬을 생성. 포털을 조사(검색 URL 패턴, 결과 구조, robots.txt/접근 규칙)하고, 배포된 것과 같은 구조로 CLI 스킬을 스캐폴딩하고, 등록 전에 실제 쿼리를 테스트한다. 인증 벽이 있는 포털은 거절되고, 제한적 약관의 포털에는 개인 사용 전용 경고가 붙는다.

`/reset`(profile/documents/all)으로 초기화할 수 있으며 삭제 전에 `RESET` 입력 확인을 요구한다.

## 국가 불문 포털 스킬

덴마크 데모 4종 외에 두 개의 country-agnostic 스킬이 함께 배포된다.

- **`linkedin-search`** — LinkedIn의 공개 비인증 `jobs-guest` 엔드포인트 기반. 런타임 의존성 0, 위치를 플래그로 받아 어느 시장에서든 동작(`-l "Berlin, Germany"` 등). 자동 접근은 LinkedIn 약관 위반이므로 **개인 사용 전용, 저볼륨** 유지.
- **`freehire-search`** — [freehire.dev](https://freehire.dev/) 애그리게이터의 공개 REST API(JSON, API 키 불요). 테크 중심, `--region`/`--country`/`--remote` 패싯 플래그로 멀티 마켓, 결과가 구조화(스킬, 시니어리티, 카테고리)되어 온다. 백엔드는 MIT 라이선스로 자가 호스팅 가능(`FREEHIRE_API_URL`).

## 더 나은 결과를 위한 팁

- **프로필 깊이가 출력 품질의 최대 변수다.** 직함만 나열하지 말고 실제 한 일(프로젝트, 도구, 측정 가능한 성과)을 서술하고, "Python"이 아니라 "scikit-learn으로 고객 이탈 예측 ML 파이프라인 구축"처럼 맥락 있는 스킬을 적어라.
- 두 가지 탐색 모드를 지원한다: 원하는 역할을 아는 **명시적 타기팅**과, 전체 이력 분석으로 생각지 못한 커리어 경로를 발굴하는 **잠재 기회 발견**. `/setup` 때 무엇이 에너지를 줬고 무엇이 소모적이었는지까지 적으면 적합도 평가에 직접 반영된다.
- 급여 벤치마킹 도구는 노조 통계, Glassdoor 익스포트 등 직접 준비한 데이터로 동작하며, 데이터가 없으면 그 단계는 건너뛴다.

> 원문: [MadsLorentzen/ai-job-search: The job search that runs on your machine.](https://github.com/MadsLorentzen/ai-job-search)
> 원본 클립: 2026-07-14-MadsLorentzenai-job-search The job search that runs on your machine. AI job application framework built on Claude Code evaluate postings, tailor CVs, write cover letters, prep interviews. Fork it and own it.
