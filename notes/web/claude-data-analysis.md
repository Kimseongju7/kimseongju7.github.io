---
title: "claude-data-analysis — Claude Code로 만드는 데이터 분석 AI 에이전트"
date: 2026-07-14 14:09:09 +0900
categories: [개발, 도구]
tags: [claude-code, data-analysis, ai-agent, subagents, slash-commands, hooks]
slug: claude-data-analysis
publish: true
---
`liangdabiao/claude-data-analysis`는 Claude Code의 서브에이전트·슬래시 커맨드·훅으로 구축한 지능형 데이터 분석 플랫폼이다. 제작자는 "Claude Code도 LangGraph, CrewAI, AutoGen 같은 에이전트 프레임워크라는 걸 잊지 말라"며, Claude Code를 개조해 데이터 분석을 채팅처럼 간단하게 만들었다고 소개한다.

## 빠른 시작

1. 데이터셋을 `data_storage/` 디렉터리에 넣는다: `cp your_data.csv ./data_storage/`
2. 슬래시 커맨드로 분석을 시작한다:

```
/analyze user_behavior_sample.csv exploratory   # 기본 탐색적 분석
/visualize user_behavior_sample.csv all          # 시각화 생성
/generate python data-cleaning                   # 분석 코드 생성
/report user_behavior_sample.csv complete markdown  # 종합 리포트 생성
```

## 핵심 구성 요소

### 서브에이전트 6종

- **data-explorer**: 통계 분석·패턴 발견 전문가
- **visualization-specialist**: 보기 좋고 인사이트 있는 차트·그래프
- **code-generator**: 프로덕션 수준의 분석 코드
- **report-writer**: 종합 분석 리포트
- **quality-assurance**: 데이터 검증·품질 관리
- **hypothesis-generator**: 연구 가설·인사이트 생성

### 슬래시 커맨드

- `/analyze [dataset] [type]` — 데이터 분석
- `/visualize [dataset] [type]` — 시각화 생성
- `/generate [language] [type]` — 분석 코드 생성
- `/report [dataset] [format]` — 리포트 생성
- `/quality [dataset] [action]` — 품질 보증
- `/hypothesis [dataset] [domain]` — 가설 생성

### 자동화 워크플로 (훅)

- 데이터 업로드 시 자동 품질 검사
- 프로젝트 컨텍스트를 인식한 분석 제안
- 문서·코드가 함께 생성되는 재현 가능한 분석

## 사용 예시

- **사용자 행동 분석**: `/analyze` → `/visualize trends` → `/quality clean` → `/report complete html` → `/generate python user-segmentation`
- **매출 분석**: `/analyze sales_data.csv statistical`, `/generate sql revenue-analysis`, `/report sales_data.csv executive pdf`
- **고객 분석**: `/analyze customer_data.csv predictive`, `/generate r clustering-analysis`, `/hypothesis customer_data churn-prediction`

## 프로젝트 구조

```
claude-data-analysis/
├── .claude/
│   ├── agents/          # 서브에이전트 설정
│   ├── commands/        # 슬래시 커맨드 정의
│   ├── hooks/           # 자동화 스크립트
│   └── settings.json    # Claude Code 설정
├── data_storage/        # 데이터 파일
├── visualizations/      # 생성된 차트
├── generated_code/      # 분석 코드
├── analysis_reports/    # 분석 리포트
├── examples/            # 예시 데이터셋·워크플로
└── docs/                # 문서
```

- 샘플 데이터 포함: `user_behavior_sample.csv` (user_id, session_id, timestamp, action, page_url, device_type, location, revenue)
- 요구 사항: Python 3.8+, 서브에이전트가 활성화된 Claude Code, CSV/JSON/Excel 데이터

## 분석·시각화·코드 생성 범위

- **분석 유형**: 탐색적(품질 평가·요약 통계·패턴 발견), 통계적(가설 검정·상관·회귀·신뢰구간), 예측적(피처 중요도·예측 모델링), 완전 분석(전체 + 리포트 + 시각화)
- **시각화 유형**: 종합 대시보드, 트렌드(시계열·이동평균), 분포(히스토그램·박스플롯·밀도), 상관([[열지도|히트맵]]·산점도), 비교(막대·그룹·스몰 멀티플)
- **코드 생성 언어**: Python(Pandas, NumPy, Scikit-learn, Matplotlib), R(Tidyverse, ggplot2, caret), SQL(주요 방언 전부), JavaScript(D3.js, Plotly.js, TensorFlow.js)

## 프로젝트 현황

- 현재 단계: Week 1.1 — 프로젝트 초기화 완료
- 완료: 프로젝트 구조·설정, data-explorer/visualization-specialist 서브에이전트, 핵심 커맨드(/analyze, /visualize, /generate), 자동화 훅, 샘플 데이터·문서
- 예정: report-writer/quality-assurance/hypothesis-generator 서브에이전트, 고급 커맨드, 인터랙티브 대시보드, 외부 도구 연동
- 라이선스 MIT. [DATAGEN](https://github.com/starpig1129/DATAGEN) 프로젝트에서 영감을 받았다.

## 관련 노트

- [[claude-code-multi-agent-sessions]] — Claude Code 서브에이전트 멀티세션 활용
- [[alirezarezvani-claude-skills]] — 데이터·분석 스킬을 포함한 대규모 스킬 라이브러리

> 원문: [liangdabiao/claude-data-analysis: Create a data analysis AI agent with Claude Code](https://github.com/liangdabiao/claude-data-analysis)
> 원본 클립: [[2026-07-14-liangdabiaoclaude-data-analysis Create a data analysis AI agent with Claude Code. Make data analysis as simple as having a chat!​​ 别忘了！claude code也是agent框架，类似langgraph,crewAI，autogen等等agent框架。我改造cl]]
