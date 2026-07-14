---
title: "claude-real-video — Claude(또는 어떤 LLM이든)가 진짜로 영상을 보게 하기"
date: 2026-07-14 14:15:07 +0900
categories: [개발, 도구]
tags: [claude, llm, video, keyframe, whisper, ffmpeg, yt-dlp, open-source]
slug: claude-real-video
publish: true
---
claude-real-video(crv)는 영상 URL이나 로컬 파일에서 장면 전환을 인식해 정말 의미 있는 프레임만 뽑고 중복을 제거한 뒤, 오디오 트랜스크립트와 함께 LLM이 읽을 수 있는 폴더로 만들어 주는 로컬 실행 도구다. 예를 들어 같은 58초 클립을 1fps 고정 샘플링하면 58프레임이지만, crv는 실제로 달라진 26프레임만 남기고 `--grid` 옵션으로 3장의 콘택트 시트에 압축한다. MIT 라이선스.

## 왜 필요한가

대부분의 AI 도구는 영상을 실제로 *보지* 못한다. ChatGPT에 YouTube 링크를 붙여넣으면 화면이 아니라 **트랜스크립트**를 읽고, Claude는 영상 파일 자체를 받지 않으며, 영상을 네이티브로 읽는 Gemini조차 Google에 업로드한 뒤 **고정 간격**(기본 1fps)으로 프레임을 샘플링해 빠른 컷은 놓친다.

crv는 처리를 전부 로컬에서 수행한다. 어딘가로 전송되는 것은 이후 사용자가 직접 LLM에 붙여넣기로 선택한 프레임/텍스트뿐이다.

```
crv "https://www.youtube.com/watch?v=..."
# → crv-out/frames/*.jpg + frames.json(프레임별 타임스탬프) + transcript.txt/.json + MANIFEST.txt
```

고정 간격 샘플링과의 비교:

| | 고정 간격 샘플링 | claude-real-video |
| --- | --- | --- |
| 프레임 선택 | N초마다 | 장면 전환 감지 + 밀도 하한 |
| 반복 샷(A-B-A 컷) | 매번 다시 전송 | 슬라이딩 윈도 dedup으로 샷당 1회 |
| 10분짜리 정적 슬라이드 | 거의 같은 프레임 ~600장 | 1장으로 압축 |
| 빠른 컷 릴 | 샘플 사이 프레임 놓침 | 시각 변화마다 포착 |
| 오디오 | 대개 무시 | Whisper 트랜스크립트(언어 자동 감지) |
| 처리 위치 | 흔히 남의 클라우드 | 내 머신 |
| 입력 | 보통 로컬 파일만 | URL(yt-dlp) 또는 로컬 파일 |

더 적고 더 의미 있는 프레임을 먹이니 컨텍스트는 싸지고 이해는 좋아진다. LLM 용도가 아니어도 ML 모델 다운로드 없는 범용 키프레임 추출기로 쓸 수 있다.

## 설치와 사용

```
pip install "claude-real-video[whisper]"   # 권장: 프레임 + dedup + 음성 전사
pip install claude-real-video              # 코어만 (프레임 + dedup)
```

- `[whisper]` extra 없이는 음성 인식(STT)이 없다(자체 자막이 있는 영상은 그래도 트랜스크립트가 생성됨).
- 시스템 요구사항: `ffmpeg`/`ffprobe` 필수 (macOS: `brew install ffmpeg`, Linux: `sudo apt install ffmpeg`, Windows: `winget install Gyan.FFmpeg` 등).
- macOS/Windows/Linux, Python 3.10+ 지원. `python -m claude_real_video`도 `crv`의 별칭으로 동작한다.

```
crv "https://www.instagram.com/reel/XXXX/"   # YouTube / Instagram / TikTok 등 링크
crv lecture.mp4 -o out --lang en              # 로컬 파일, 영어 전사, ./out 출력
crv clip.mp4 --no-transcribe                  # 프레임만
crv "https://..." --cookies cookies.txt       # 로그인 필요한 영상(본인의 인가된 접근)
```

Python API도 있다: `from claude_real_video import process`.

## 주요 옵션

- `--scene`(기본 0.30) 장면 전환 민감도, `--fps-floor`(기본 1.0) 최소 N초당 1프레임, `--max-frames`(기본 150) 프레임 상한
- `--adaptive` — 느리게 변하는 콘텐츠(애니메이션 튜토리얼, 점진적 변형, 느린 팬)용. 고정 임계값 대신 롤링 이웃 대비로 프레임을 골라 2~3초짜리 스쿼시-앤-스트레치도 잡는다.
- `--text-anchors` — 텍스트 위주 콘텐츠(강의 슬라이드, 화면 녹화)용. 자막 큐 타임스탬프에 프레임을 강제 추가한다. 사이드카 `.srt`/`.vtt` 또는 내장 자막 트랙 필요(픽셀에 박힌 자막은 감지 불가), 초당 최대 1프레임.
- `--dedup-threshold`(기본 8) 새 프레임으로 치려면 바뀌어야 하는 픽셀 비율(%), `--dedup-window`(기본 4) 최근에 유지된 N프레임과 비교
- `--report` — 버린 프레임을 `./dropped`에 보존하고 keep/drop 결정 전부를 시각화한 `report.html` 생성(튜닝용)
- `--keep-audio` — 전체 사운드트랙(`audio.m4a`)도 저장해 오디오를 들을 수 있는 모델(Gemini, GPT-4o 등)이 음악·톤까지 듣게 함
- `--viewer` — 영상 + 키프레임 그리드 + 트랜스크립트를 한 로컬 페이지로 보는 `viewer.html` 생성(네트워크 불필요)
- `--grid` — 유지된 프레임을 3x3 콘택트 시트로 타일링. 연속 프레임이 나란히 있어 모델이 움직임과 진행을 따라가기 좋다.
- `--why`(0.3.0 신규) — 시청 목적을 지정(예: `--why "find the pricing strategy"`). MANIFEST.txt에 기록되어 일반 요약 대신 그 관점으로 분석하게 한다.
- `--kb`(0.3.0 신규) — 분석 결과를 지정 폴더(Obsidian 볼트 등)에 날짜 붙은 마크다운 노트로 저장해 `crv-out`에서 죽지 않게 한다.
- `--cookies-from-browser` — chrome/safari/firefox/edge 브라우저에서 직접 로그인 쿠키를 읽음(본인 계정 한정)
- `--overwrite` — 이전 분석이 있는 출력 폴더를 교체(없으면 영상이 섞이지 않도록 비어 있지 않은 폴더를 거부)

터미널 없이 쓰려면 `crv-web`을 실행하면 로컬 페이지(번체/간체 중국어·영어)가 열린다. 링크나 파일 경로를 붙여넣고 Analyze를 누르면 되고, 원본 영상은 업로드되지 않는다.

## Claude Code 스킬로 설치

스킬로 설치하면 Claude Code에 영상 링크만 붙여넣어도 알아서 영상을 본다(`skills/` 폴더는 pip 패키지가 아닌 레포에 있으므로 클론 필요).

```
pip install "claude-real-video[whisper]"
git clone https://github.com/HUANGCHIHHUNGLeo/claude-real-video.git
mkdir -p ~/.claude/skills && cp -r claude-real-video/skills/claude-real-video ~/.claude/skills/
```

## 동작 원리

1. **Fetch** — URL은 `yt-dlp`(쿠키 선택), 로컬 파일은 복사
2. **Extract** — `ffmpeg select` 한 번의 시간순 패스로 모든 장면 전환 + 밀도 하한(`--fps-floor`)의 프레임 확보. 빠른 컷과 느린 스크린캐스트 모두 커버.
3. **Dedup** — 최근 유지 프레임의 슬라이딩 윈도에 대해 두 개의 감지기를 돌린다. *글로벌* 채널은 실제 픽셀 차이를 측정(퍼셉추얼 해시가 아닌 다운스케일 RGB — 해시는 단색·동일 휘도 색상 변화에 취약)하고, *settled-local* 채널(v0.7.4)은 전역 평균으로는 ~0%라 안 잡히는 얇은 펜 획, 자막 카드 교체, 작은 UI 변화를 잡는다(1px 이동 허용, 쿨다운으로 깃발 펄럭임 같은 반복 모션 오탐 방지). 마지막 프레임은 움직이는 중이어도 평가해 영상의 종료 상태를 놓치지 않는다.
4. **Text** — 영상에 이미 자막(사이드카 `.srt`/`.vtt` 또는 내장 트랙)이 있으면 그것을 트랜스크립트로 사용(재전사보다 빠르고 정확). 없을 때만 Whisper로 폴백. `--whisper-model turbo`는 large-v2에 가까운 정확도를 약 8배 속도로 낸다(1.6GB 1회 다운로드, 메모리 ~6GB).
5. **Audio**(선택) — `--keep-audio`로 원본 사운드트랙 전체 저장
6. **Timestamps** — 유지된 모든 프레임의 원본 영상 시각이 파이프라인 전체를 거쳐 `frames.json`(`file`/`timestamp_sec`/`timestamp`/`selection_reason`)에 기록된다. `frame_012 @ 00:03:41`처럼 시각 증거를 인용하거나 `transcript.json` 세그먼트와 정렬하거나 비디오 RAG 파이프라인에 먹일 수 있다.
7. **Manifest** — `MANIFEST.txt`가 모델을 위해 전체를 요약

결과적으로 모델은 영상을 **보고**(키프레임), **읽고**(트랜스크립트), `--keep-audio`가 있으면 **듣는다**(사운드트랙).

## 유료 애드온 crv Pro

무료 버전은 화면에 *무엇이* 있는지 알려주고, crv Pro는 영상이 *어떻게* 찍혔는지 알려준다. PyPI 패키지 약칭은 crv이며, 유료 애드온은 Capafy에서 "llm-real-video Pro"라는 이름으로 판매된다(파운더 가격 일시불 $19).

- 카메라 무브 분류(static/pan/tilt/zoom/handheld), 편집 리듬(샷 리스트, 분당 컷 수, 페이싱 변화)
- 지각 타임라인: 프레임이 못 담는 제스처·표정·음성 피치 변화·화자 감정·비음성 사운드 이벤트를 타임스탬프와 함께
- `--breakdown` 리포트: 훅 분석, 페이싱 커브, 카메라 언어, Reels 알고리즘 관점 + LLM이 채워 완성하는 루브릭
- 3가지 모드: `--mode watch`(내용 이해) / `--mode creator`(제작 역설계) / `--mode full`
- 2026년 7월 업데이트: 음악 상태 타임라인(BPM 포함), 분리된 보이스에서 읽는 음성 감정, 클릭 가능한 동기화 타임라인의 인터랙티브 `--viewer` 대시보드 등

## 주의사항

- 권리가 있는 콘텐츠만 다운로드할 것. `--cookies`는 본인의 인가된 접근용이며 자격증명을 레포에 넣지 말 것.
- 영상 하나당 출력 폴더 하나를 사용할 것.

## 관련 노트

- [[headroom-context-compression]] — LLM에 도달하기 전 컨텍스트를 줄여 비용을 아끼는 접근(더 적은 프레임 = 싼 컨텍스트)
- [[prompting-claude-fable-5]] — 밀도 높은 기술 이미지·스크린샷을 해석하는 비전 능력

> 원문: [HUANGCHIHHUNGLeo/claude-real-video: Let Claude (or any LLM) actually watch a video](https://github.com/HUANGCHIHHUNGLeo/claude-real-video)
> 원본 클립: [[2026-07-14-HUANGCHIHHUNGLeoclaude-real-video Let Claude (or any LLM) actually watch a video — scene-aware, deduplicated frames + transcript, from a URL or local file. Runs locally, MIT.]]
