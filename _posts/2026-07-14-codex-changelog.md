---
title: 'Codex 체인지로그 — 2026년 6~7월 릴리스 하이라이트'
date: 2026-07-14 14:38:08 +0900
categories: [개발, 도구]
tags: [codex, openai, changelog, cli, chatgpt, release-notes]
description: 'OpenAI Codex 공식 체인지로그 중 2026년 6~7월 CLI·앱·서비스 릴리스 하이라이트 정리.'
---
OpenAI Codex의 공식 체인지로그 페이지 정리. 원문은 Codex CLI 0.144.3부터 2025년 5월까지 거슬러 올라가는 방대한 릴리스 기록(CLI, Codex 앱, ChatGPT iOS 앱, 서비스 공지 포함)이며, 여기서는 최근(2026년 6~7월) 하이라이트를 중심으로 요약한다.

## Codex CLI 0.144.x (2026-07-09~)

- **0.144.3** — 버전 번호만 올린 릴리스(rust-v0.144.2 이후 병합된 변경 없음).
- **0.144.2** — 프롬프팅 회귀를 롤백하고 기존 Guardian 자동 리뷰 정책·요청 형식·툴 동작을 복원 (#32672).
- **0.144.1** — GitHub이 압축·재정렬된 릴리스 메타데이터를 반환할 때 독립 설치가 실패하던 문제 수정, macOS 패키지 설치가 `codex` 실행 파일과 함께 code-mode 호스트를 노출하도록 보장, 동반 호스트 바이너리가 없으면 내장 런타임으로 폴백해 code mode 유지 (#31913).
- **0.144.0** 주요 신기능:
  - 사용량 한도 리셋 크레딧에 종류·만료가 표시되고 어떤 크레딧을 사용할지 선택 가능 (#30488)
  - 선언된 읽기 전용 동작은 허용하고 쓰기에는 확인을 요구하는 `writes` 앱 승인 모드 추가 (#30482)
  - MCP 툴이 실험 옵트인 없이 대화형 인증 요청 가능 (#28772)
  - app-server 호스트가 런타임에 Codex 인증을 제공하고 로그인 성공 시 호스팅 페이지로 리디렉션 가능 (#28745, #31274)
  - Ultra 리즈닝 선택 시 높은 멀티에이전트 동시성이 사용량을 급증시킬 수 있다는 경고 표시 (#31621)
  - 주요 수정: 은퇴한 모델을 참조하는 컴팩션으로 재개 실패하던 ChatGPT 스레드 복구, Intel macOS Code Mode 크래시 수정, Windows 샌드박스의 쓰기 가능 루트 내 파일 삭제 허용, 터미널 제어 시퀀스 붙여넣기로 인한 TUI 깨짐 방지, 시스템 프록시·커스텀 CA를 존중하는 저지연 WebSocket 유지 등

## Codex, ChatGPT 데스크톱 앱에 통합

Codex가 macOS·Windows용 ChatGPT 데스크톱 앱의 일부가 됐다. 기존 Codex 앱 사용자는 평소처럼 업데이트하면 프로젝트·설정·워크플로가 유지되고, Codex를 기본 뷰로 지정할 수 있으며 macOS에서는 Codex 앱 아이콘도 유지 가능하다.
- 신기능: 앱에서 Markdown·코드 직접 편집과 인라인 주석, 선택한 내용 수정 요청 / 사이드바에서 리뷰어 피드백과 diff를 나란히 보는 GitHub PR 리뷰 / 한 프로젝트에서 여러 저장소 작업.
- 개선: GPT-5.6으로 Computer Use 고속화, 작업 진행 상황 가독성 개선, 플러그인 관리의 Settings 통합, 모바일 연결 안정성과 SSH 프로젝트 비디오 렌더링 수정.

## Codex CLI 0.143.0 (2026-07-08)

- 원격 플러그인 기본 활성화(풍부한 카탈로그 행, npm 마켓플레이스 소스, 원격/로컬 버전 표시)
- macOS·Windows 시스템 프록시(PAC, WPAD 포함)로 인증·Responses API 트래픽 라우팅
- 실행 중인 데몬에서 수동 페어링 코드를 만드는 `codex remote-control pair` 추가
- Amazon Bedrock GPT-5.6 Sol·Terra·Luna 모델 추가, `max` 리즈닝 에포트 1급 지원
- MCP 툴이 기본으로 tool search 사용, ChatGPT 호스티드 MCP 서버의 세션 인증 명시 사용 가능
- 수정: Windows ConPTY 입력(줄바꿈·백스페이스), 오래된 TUI 안전 프롬프트, GitHub API 레이트리밋으로 인한 설치 실패 감소 등. OpenSSL·Hono·fast-uri·quick-xml·crossbeam-epoch 보안 업데이트.

## ChatGPT iOS 앱의 Codex 기능 (2026-06-15 / 06-22 / 07-06)

- **1.2026.181 (7/6)**: 대화에서 직접 Codex 태스크 생성·검색·열기·포크·관리, staged/unstaged/브랜치/마지막 턴 변경 필터와 브랜치 비교, 트랜스크립트 선택 텍스트를 컴포저에 추가, 첨부 미리보기와 인라인 사진·카메라 선택기, 개인 키 또는 무자격 증명 SSH 호스트 지원, 태스크 메뉴의 사용량 한도·크레딧 표시.
- **1.2026.167 (6/22)**: 호스트별 성격 설정(Friendly/Pragmatic), 컴포저에서 목표(goal) 직접 편집, 포크된 대화에서 원본 스레드로 돌아가는 링크.
- **1.2026.160 (6/15)**: 워크스페이스 파일 브라우저와 디렉터리 선택기, diff 전체 펼치기/접기, MCP 승인 범위 선택(현재 채팅/전체 채팅), 메시지·플랜의 LaTeX 렌더링.

## Codex Remote 정식 출시(GA, 2026-06-25)

ChatGPT 모바일 앱에서 연결된 Mac·Windows 호스트의 작업을 시작·계속하고, 진행 상황을 검토하고, 승인 요청을 폰에서 처리할 수 있다. Remote Control은 이제 iOS/Android 기기와 호스트 간 인증된 1:1 QR 페어링을 사용한다(2026년 6월 8일 이후 사용된 연결은 유지, 그 이전의 비활성 연결은 재페어링 필요). 새 DigitalOcean 플러그인으로 Codex가 Droplet을 [프로비저닝](/posts/provisioning/)하고 SSH를 구성해 원격 워크스페이스로 연결할 수 있다.

## Codex CLI 0.142.x (2026-06-22~07-01)

- **0.142.0**: `/usage`에서 사용량 리셋 크레딧 확인·사용, `/plugins`의 OpenAI Curated/Workspace/Shared with me 섹션 구성과 턴 중 플러그인 추천·설치, 에이전트 스레드 전반의 롤아웃 토큰 예산 설정(잔여 예산 알림·소진 시 턴 중단), 멀티에이전트 위임을 스레드·턴 수준에서 비활성/명시 요청만/능동적으로 설정, 서버 승인 URL만 직접 접근을 허용하는 인덱스드 웹 검색 모드, UTC 시간 알림 수신과 현재 시각 조회 기능.
- **0.142.1~0.142.2**: Windows/macOS 시스템 프록시(PAC·WPAD) 지원, MCP tool search 기본화, 플러그인 다크모드 로고, Bedrock 만료 자격증명에 실행 가능한 복구 안내, 안전 분류기가 검사 못 하는 PowerShell AST 영역 실행 시 승인 요구 등.
- **0.142.5**: Responses WebSocket 요청 페이로드 전체가 트레이스 로그에 기록되던 문제 방지 (#30771). 0.142.3/0.142.4는 사용자 영향 없는 유지보수 릴리스.

## Codex 앱 26.616과 0.141.0, 지역 확대 (2026-06-16~18)

- **Codex 앱 26.616**: 시연한 워크플로를 재사용 가능한 스킬로 바꾸는 macOS **Record & Replay**(EEA·영국·스위스 제외로 시작, Computer Use 활성화 필요), 자동화 실행 기록의 일괄 처리, 로컬·원격 호스트 간 **스레드 핸드오프**(Codex가 핸드오프를 조율 가능), SSH 연결 관리 딥링크.
- **CLI 0.141.0**: 원격 실행기의 인증·종단간 암호화 Noise 릴레이 채널, 크로스 플랫폼 원격 실행의 네이티브 경로·셸 보존, 스레드별 실행기 플러그인 stdio MCP 활성화, TUI 입력 프롬프트의 비활성 시 자동 해결(카운트다운), WAL 리셋 손상 수정판 SQLite 고정, 엔터프라이즈 프록시용 P-521 인증서 지원 등.
- **6/16**: EEA·영국·스위스에 Computer Use(macOS·Windows), Codex Chrome 확장, Memories(해당 지역 기본 꺼짐), Chronicle(macOS ChatGPT Pro 옵트인 리서치 프리뷰) 롤아웃.

## Codex CLI 0.140.0 (2026-06-15)

- `/usage`의 일간·주간·누적 계정 토큰 사용량 뷰
- `codex delete`, `/delete`, app-server `thread/delete`를 통한 영구 세션 삭제(확인 절차와 서브에이전트 정리 포함)
- **Claude Code에서 설정·프로젝트 구성·최근 채팅을 선택적으로 가져오는 `/import`**
- `@` 입력 시 파일·플러그인·스킬 통합 멘션 메뉴 기본 열림
- Bedrock API 키 관리 인증, CLI·MCP OAuth 자격증명의 암호화 로컬 저장
- 손상된 SQLite 상태 DB의 자동 백업·재구축 등 안정성 수정

## 그 외

체인지로그는 이 앞으로도 2026년 상반기~2025년(0.139.0, 0.138.0 릴리스, 각종 앱 업데이트, 2025-05-22의 초기 공지, ChatGPT iOS 앱의 Codex 지원 등)까지 이어진다. 상세 PR 목록은 각 릴리스의 GitHub 릴리스 페이지(`openai/codex`)에서 확인할 수 있다.

## 관련 노트

- [codex-top-6-skills](/posts/codex-top-6-skills/) — Codex 에이전트를 확장하는 대표 스킬 6선
- [openai-codex-plugin-cc](/posts/openai-codex-plugin-cc/) — Codex용 OpenAI 공식 플러그인
- [harness-engineering-codex](/posts/harness-engineering-codex/) — Codex 하네스 엔지니어링
- [gpt-5-6-sol-in-claude-code](/posts/gpt-5-6-sol-in-claude-code/) — Claude Code에서 GPT-5.6 Sol 사용

> 원문: [Codex changelog | ChatGPT Learn](https://learn.chatgpt.com/docs/changelog#codex-2026-06-18-app)
> 원본 클립: 2026-07-14-Codex changelog  ChatGPT Learn
