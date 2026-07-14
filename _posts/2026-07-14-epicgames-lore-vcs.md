---
title: 'EpicGames/lore — 에픽게임즈의 차세대 오픈소스 버전 관리 시스템'
date: 2026-07-14 14:33:59 +0900
categories: [개발, 도구]
tags: [lore, epic-games, version-control, vcs, git, open-source, merkle-tree]
description: '에픽게임즈가 공개한 대용량 바이너리 에셋과 코드를 함께 다루는 차세대 오픈소스 버전 관리 시스템 Lore 정리.'
---
에픽게임즈(Epic Games)가 공개한 **Lore**는 코드와 대용량 바이너리 에셋을 함께 다루는 프로젝트(게임·엔터테인먼트)를 위해 설계된 차세대 오픈소스 버전 관리 시스템이다. 데이터와 팀 규모 양쪽에서 전례 없는 확장성을 목표로 하며, 개발자와 아티스트의 요구를 모두 겨냥한다.

## 개요

- 대용량 바이너리 에셋 + 코드 조합에 최적화된 중앙집중형 버전 관리 시스템.
- 현재 pre-1.0 상태로 활발히 개발 중이라 인터페이스, 온디스크 포맷, API가 릴리스 간에 바뀔 수 있다.
- MIT 라이선스로 완전 오픈소스이며, 오픈 스탠더드 기반의 열린 생태계를 지향한다.
- UEFN(Unreal Editor for Fortnite)의 내장 버전 관리 시스템이기도 하다. 다만 UEFN 빌드는 오픈소스 프로젝트에 포함할 수 없는 독점 압축 포맷을 쓰고 있어 아직 오픈소스 툴링과 호환되지 않으며, 에픽은 UEFN을 오픈 압축 포맷(오픈소스 프로젝트와 동일한 포맷)으로 이전해 이 간극을 없애는 작업을 진행 중이다.

## 주요 특징

- **쉬운 시작과 [온디맨드](/posts/on-demand/) 확장** — 로컬 모드로 몇 분 만에 시작하고, 필요한 만큼 빠르게 확장.
- **빠르고 효율적인 프로세스** — 공유·재사용 가능한 데이터와 필요 시점 다운로드 덕분에 규모가 커져도 느려지지 않음.
- **자유로운 브랜칭** — 브랜치를 빠르게 만들고 관리·동기화하며 자유롭게 실험·반복·릴리스.
- **신뢰할 수 있는 히스토리** — 변조 여부를 검증할 수 있는(tamper-evident) 단일 진실 소스로 리비전을 추적.
- **직관적 인터페이스** — CLI로 Lore의 전체 기능에 1:1로 접근.
- **전체 표면 API** — C/C++, C#, Rust, Go, Python, JavaScript로 확장·커스터마이즈·통합 가능.

## 아키텍처

콘텐츠 주소 방식(content-addressed)의 중앙집중형 시스템으로, 저장소 상태를 머클 트리(Merkle tree)와 불변 리비전 체인으로 표현한다. 바이너리 우선 저장, 중복 제거, 대규모 희소(sparse)/온디맨드 데이터 하이드레이션에 최적화되어 있다.

- **콘텐츠 주소 저장** — 데이터를 콘텐츠 해시로 저장·참조해 빠른 비교, 무결성 검사, 히스토리·브랜치 간 재사용이 가능.
- **불변 리비전 체인** — 리비전 해시가 부모 리비전 해시와 포함 데이터 해시에서 파생되어 암호학적 무결성을 가진 불변 체인을 형성.
- **대용량 파일 청크 저장** — 파일을 재사용 가능한 청크로 저장하고 인덱스로 조회해 중복을 줄이고 대형 바이너리 에셋의 업데이트·전송을 효율화.
- **온디맨드 하이드레이션과 희소 워크스페이스** — 필요한 시점에만 파일 데이터를 가져와 워크스페이스를 가볍게 유지. 전부 미리 내려받을 필요가 없다.
- **캐싱을 갖춘 중앙집중 서비스** — 내구성 스토리지 앞단의 캐싱으로 대규모 팀·저장소의 처리량을 확장.
- **경량 브랜치와 빠른 전환** — 브랜치는 가벼운 가변 참조라서 생성·전환 오버헤드가 낮고 데이터 중복이 없다.

## 시작하기

데모 모드로 로컬 서버를 바로 띄울 수 있다.

macOS / Linux:

```
curl -fsSL https://raw.githubusercontent.com/EpicGames/lore/main/scripts/install.sh | bash -s -- --demo
```

Windows (PowerShell):

```
$env:LORE_DEMO=1; irm https://raw.githubusercontent.com/EpicGames/lore/main/scripts/install.ps1 | iex
```

- 퀵스타트, 문서, FAQ(라이선스·지원 플랫폼·프로덕션 준비도·타 VCS 비교), 로드맵(확장 가능한 락킹부터 오픈소스 데스크톱 클라이언트까지)이 공식 문서 사이트에 정리되어 있다.

## 저장소 구성

핵심 라이브러리·서버·CLI가 메인 저장소(EpicGames/lore)에 있고, 언어별 SDK가 별도 저장소로 제공된다.

| 저장소 | 설명 |
| --- | --- |
| Lore Library, Server & CLI | 핵심 라이브러리 + 서버 + CLI (메인 저장소) |
| lore-js | JavaScript SDK |
| lore-python | Python SDK |
| lore-dotnet | C# SDK |
| lore-go | Go SDK |

코드·문서·버그 리포트·리뷰 등 모든 형태의 기여를 환영하며, 입문자는 `good-first-issue` 라벨부터 시작하면 된다.

## 관련 노트

- [브랜치, 분기의 정책 고민](/posts/branch-policy/) — 브랜칭 전략을 고민한 노트와 이어지는 버전 관리 주제
- [zilliz-claude-context](/posts/zilliz-claude-context/) — 머클 트리 기반 증분 인덱싱을 쓰는 또 다른 사례(콘텐츠 주소·변경분만 처리)

> 원문: [EpicGames/lore: Lore is a next-generation, open source version control system](https://github.com/EpicGames/lore)
> 원본 클립: 2026-07-14-EpicGameslore Lore is a next-generation, open source version control system
