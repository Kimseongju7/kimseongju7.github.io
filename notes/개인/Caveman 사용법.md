---
title: Caveman 사용법
date: 2026-07-26
categories: [기록, 도구]
tags: [caveman, claude-code, plugin, tokens, hooks]
slug: caveman-usage
publish: true
---

[[caveman-token-compression|Caveman]] 플러그인을 실제로 쓰면서 정리한 설정·운영 요령. 무엇인지에 대한 설명은 정리본 노트에 있고, 여기는 **내 환경에서 어떻게 켜고 끄는가**만 다룬다.

## 한눈에

- 압축되는 건 **대화 응답 스타일**뿐. 파일에 쓰는 내용(노트 본문, 코드, 커밋 메시지)은 원래 톤 그대로다.
- 플러그인 훅으로 동작하므로 **프로젝트를 안 가린다**. 터미널이든 옵시디언에서 띄운 실행이든 똑같이 걸린다.
- 기본값은 설정 파일로 바꿀 수 있고, 프로젝트마다 따로 지정할 수 있다.

## 강도(mode)

| 값 | 무엇이 달라지나 |
| --- | --- |
| `off` | 비활성. 훅이 규칙 자체를 주입하지 않는다 |
| `lite` | filler·hedging만 제거. 관사와 완전한 문장은 유지. 실무적이고 담백한 톤 |
| `full` | 기본값. 관사 생략, 문장 조각 허용, 짧은 동의어. 툴 호출 설명·장식용 표·이모지 없음 |
| `ultra` | 인과가 모호해지지 않는 선에서 접속사까지 제거. 한 단어로 되면 한 단어 |

이 외에 `wenyan-lite` / `wenyan` / `wenyan-full` / `wenyan-ultra`(문언문 압축)와, 단발성 모드인 `commit` / `review` / `compress`가 있다.

## 켜고 끄기

**세션 중 임시로** — 슬래시 커맨드. 기본값이 `off`여도 커맨드는 살아 있다.

```
/caveman full
/caveman lite
/caveman off
```

자연어로도 해제된다: `normal mode`, `stop caveman`.

**전역 기본값** — `~/.config/caveman/config.json`

```json
{
  "defaultMode": "off"
}
```

**프로젝트별 기본값** — 그 레포 루트에 두면 팀원 환경을 건드리지 않고 고정된다.

```bash
mkdir -p .caveman && printf '{"defaultMode":"full"}\n' > .caveman/config.json
```

`.caveman/config.json` 대신 `.caveman.json` 한 파일로 둬도 된다.

### 우선순위

`CAVEMAN_DEFAULT_MODE` 환경변수 → `<cwd>/.caveman/config.json` 또는 `.caveman.json`(cwd에서 위로 올라가며 첫 매치) → `~/.config/caveman/config.json` → `full`

## 내 설정

전역을 `off`로 두고, caveman이 필요한 코드 레포에만 `.caveman/config.json`으로 켜는 방식.

이유는 이 vault의 주 작업이 **글쓰기**라서다. 노트·블로그 글은 압축된 말투가 도움이 안 되고, 톤을 잡거나 문장을 다듬을 때 예시 문장까지 caveman으로 나오면 오히려 헷갈린다. 반대로 코드 작업은 답이 짧을수록 읽기 편하니 레포 단위로 켜는 게 맞다.

## 주의할 점

- **훅은 프로젝트 스코프가 아니다.** `enabledPlugins`에 등록된 플러그인이 `SessionStart`와 `UserPromptSubmit`에 훅을 걸기 때문에, 옵시디언 버튼으로 스킬을 돌려도 그대로 적용된다. 터미널에서만 걸릴 거라고 생각했는데 아니었다.
- **세션 중 변경은 그 세션에만 남는다.** `/caveman off`를 해도 다음 세션은 다시 기본값으로 시작한다. 영구히 바꾸려면 설정 파일을 고쳐야 한다.
- **자동으로 압축이 풀리는 구간이 있다.** 보안 경고, 되돌릴 수 없는 작업 확인, 순서를 잘못 읽으면 위험한 다단계 절차에서는 caveman이 꺼진다. 압축 자체가 모호함을 만드는 경우도 마찬가지.

## 딸린 기능

- `/caveman-commit` — Conventional Commits 형식의 압축된 커밋 메시지. 제목 50자 이하, 본문은 "왜"가 자명하지 않을 때만
- `/caveman-review` — 한 줄에 하나씩(위치·문제·수정)인 코드 리뷰 코멘트
- `/caveman-stats` — 현재 세션의 실제 토큰 사용량과 절감량. 훅이 세션 로그에서 직접 읽어 출력한다(모델 추정 아님)
- `/caveman-compress FILEPATH` — `CLAUDE.md` 같은 메모리 파일을 압축해 **입력 토큰**을 줄인다. 원본은 `FILE.original.md`로 백업
- **cavecrew 서브에이전트** — `cavecrew-investigator`(코드 위치 찾기), `cavecrew-builder`(1~2 파일 수정), `cavecrew-reviewer`(diff 리뷰). 서브에이전트 출력이 압축돼 돌아오므로 메인 컨텍스트가 덜 닳는다

## 관련

- [[caveman-token-compression]] — Caveman이 무엇인지, 설치와 벤치마크
- [[블로그 시작]]
