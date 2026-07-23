---
title: 시작하기
tags: [meta]
publish: false
---
# 시작하기

이 폴더(`notes/`)는 옵시디언에서 글을 쓰는 공간입니다. 여기서 자유롭게 `[[위키링크]]`로 노트를 연결하면 옵시디언 그래프가 자라납니다.

## 워크플로우

1. **오늘 일지 만들기** — `/today` (또는 옵시디언 Shell Commands 버튼)
2. **거친 메모 정리** — `/format-entry` 로 표준 일지 포맷으로 정리
3. **관계 맺기** — `/auto-link` 로 관련 노트에 `[[링크]]`와 태그 자동 추가
4. **카테고리 정리** — `/organize-categories` 로 태그/카테고리 점검
5. **발행** — front matter에 `publish: true` 추가 후 `/publish` → `_posts/`로 변환 + 커밋·푸시

## 규칙

- `notes/`가 원본(canonical), `_posts/`는 발행 스킬이 만드는 생성물입니다. `_posts/`를 직접 수정하지 마세요.
- 한글 제목 글을 발행할 땐 front matter에 ASCII `slug:` 를 넣으면 URL이 깔끔합니다.
- 카테고리는 최대 2단계 `[상위, 하위]`, 태그는 소문자입니다.

관련: [[daily-template]]

## 관련 노트

- [[블로그 시작]] — 이 볼트와 워크플로(스킬 5개)를 처음 구축한 날의 기록
- [[슬러그]] — 한글 제목 글 발행 시 front matter에 넣는 ASCII slug 규칙의 용어 노트
