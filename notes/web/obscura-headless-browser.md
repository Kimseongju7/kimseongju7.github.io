---
title: "Obscura — AI 에이전트와 웹 스크래핑을 위한 Rust제 헤드리스 브라우저"
date: 2026-07-14 14:04:42 +0900
categories: [개발, 도구]
tags: [headless-browser, rust, web-scraping, ai-agent, cdp, puppeteer, playwright, mcp, open-source]
slug: obscura-headless-browser
publish: true
---
Obscura는 Rust로 작성된 오픈소스 헤드리스 브라우저 엔진이다. V8로 실제 JavaScript를 실행하고 Chrome DevTools Protocol(CDP)을 지원해, Puppeteer/Playwright에서 헤드리스 Chrome의 드롭인 대체재로 쓸 수 있다 — 메모리 30MB, 페이지 로드 85ms로 Chrome보다 훨씬 가볍고 빠르며 안티 디텍트가 내장돼 있다.

## 헤드리스 Chrome 대비 장점

데스크톱 브라우징이 아니라 대규모 자동화를 위해 설계됐다.

| 항목 | Obscura | 헤드리스 Chrome |
| --- | --- | --- |
| 메모리 | **30 MB** | 200+ MB |
| 바이너리 크기 | **70 MB** | 300+ MB |
| 안티 디텍트 | **내장** | 없음 |
| 페이지 로드 | **85 ms** | ~500 ms |
| 기동 | **즉시** | ~2s |
| Puppeteer / Playwright | **지원** | 지원 |

벤치마크(정적 HTML 51ms vs ~500ms, JS+XHR+fetch 84ms vs ~800ms, 동적 스크립트 78ms vs ~700ms)는 별도 저장소 `h4ckf0r0day/obscura-benchmark`에 전체 스위트(WPT 적합성, 장애물 코스, 실사이트 코퍼스, Chrome 대비 속도)가 있다.

## 프로젝트 현황

- GitHub 스타 10,000개 달성. 호스팅 버전 **Obscura Cloud**(관리형 인프라, 레지덴셜 프록시, 전담 지원)를 준비 중이다.
- 오픈소스 엔진은 Apache-2.0으로 유지되며, 기능 제한(feature gating)은 앞으로도 없다고 명시.
- 여러 프록시 업체(SX.org, ProxyEmpire, MangoProxy, 9Proxy, NodeMaven, Rapidproxy)의 스폰서십으로 독립 개발을 유지한다.

## 설치

- **바이너리 다운로드**: Releases에서 Linux x86_64/ARM64, macOS Apple Silicon/Intel, Windows용 아카이브 제공. Arch Linux는 AUR(`yay -S obscura-browser`). Chrome·Node.js·의존성 불필요. 아카이브에는 `obscura`와 `obscura-worker`가 함께 들어 있으며 병렬 `scrape` 명령을 쓰려면 같은 디렉터리에 둬야 한다. Linux 빌드는 Ubuntu 22.04(glibc 2.35+) 타깃.
- **Docker**: `docker run -d --name obscura -p 127.0.0.1:9222:9222 h4ckf0r0day/obscura` — `distroless/cc` 기반 멀티스테이지 빌드, 셸·패키지 매니저 없음, 압축 약 57MB.
- **소스 빌드**: Rust 1.75+ 필요, `cargo build --release`(스텔스 모드는 `--features stealth`). 첫 빌드는 V8 소스 컴파일 때문에 약 5분(이후 캐시).

## 기본 사용법

- 단일 페이지: `obscura fetch <URL>` 에 `--eval`(JS 식 평가), `--dump`(html / text / links / markdown / assets / original), `--output`(파일 저장), `--wait-until networkidle0`, `--timeout`, `--selector`, `--proxy`(HTTP/SOCKS5) 등을 조합.
  - `--dump original`은 JS/DOM 레이어를 우회해 원시 응답 바디를 바이너리 안전하게 스트리밍 — 이미지·JSON·JS·CSS 등 비HTML 리소스용.
  - `--dump assets`는 페이지가 가져올 모든 서브 리소스 URL을 NDJSON으로 나열.
- CDP 서버: `obscura serve --port 9222` (+`--stealth`, `--workers N`, `--obey-robots`).
- 병렬 스크래핑: `obscura scrape url1 url2 ... --concurrency 25 --eval "..." --format json` — 워커들은 전역 프록시를 상속하고, `--quiet`로 진행 로그를 끌 수 있다.

## Puppeteer / Playwright 연동

- Puppeteer: `puppeteer-core`로 `browserWSEndpoint: 'ws://127.0.0.1:9222/devtools/browser'`에 connect.
- Playwright: `playwright-core`의 `chromium.connectOverCDP({ endpointURL: 'ws://127.0.0.1:9222' })`.
- 폼 제출 시 POST 처리, 302 리다이렉트 추적, 쿠키 유지까지 Obscura가 처리한다.

## 스텔스 모드

`--features stealth` 빌드에서 활성화.

- **안티 핑거프린팅**: 세션별 핑거프린트 랜덤화(GPU, 화면, 캔버스, 오디오, 배터리), 사실적인 `navigator.userAgentData`(Chrome 145, high-entropy), 디스패치 이벤트의 `event.isTrusted = true`, 내부 프로퍼티 은닉, 네이티브 함수 마스킹(`Function.prototype.toString()` → `[native code]`), `navigator.webdriver = undefined`.
- **트래커 차단**: 3,520개 도메인 차단(분석·광고·텔레메트리·핑거프린팅 스크립트), `--stealth`와 함께 자동 활성화.

## CDP API와 튜닝

- 구현 도메인: Target, Page, Runtime, DOM, Network, Fetch(라이브 인터셉션), IO(청크 스트리밍), Storage, Input, 그리고 DOM→Markdown 변환용 `LP.getMarkdown`.
- 대용량 응답은 `Fetch.takeResponseBodyAsStream` + `IO.read`/`IO.close`로 청크 스트리밍. 캐시 한도(`OBSCURA_NETWORK_BODY_BUFFER_BYTES`, 기본 2MiB) 초과 바디는 보존되지 않으므로 대용량 다운로드 시 한도를 올릴 것.
- V8 튜닝: `--v8-flags "--max-old-space-size=4096"`처럼 V8 플래그를 그대로 전달(주로 JS 힙 부족 해결).
- 무거운 SPA: 스크립트 실행 예산 기본 30초. 느린 네트워크의 대형 React/Vue/Angular SPA는 `OBSCURA_SCRIPT_DEADLINE_MS=60000`처럼 예산을 올리고 CDP 클라이언트의 내비게이션 타임아웃도 맞춰준다.

## MCP 서버

AI 에이전트(Claude Desktop, Cursor 등)에 브라우저 자동화 도구를 노출하는 MCP 서버를 내장한다.

- 기동: `obscura mcp`(stdio, 기본) 또는 `obscura mcp --http --port 8080`(HTTP, 엔드포인트 `/mcp`). 옵션: `--proxy`, `--user-agent`, `--stealth`.
- 제공 도구: `browser_navigate`, `browser_snapshot`(URL·제목·본문 텍스트), `browser_click`, `browser_fill`, `browser_type`, `browser_press_key`, `browser_select_option`, `browser_evaluate`, `browser_wait_for`, `browser_network_requests`, `browser_console_messages`, `browser_close`.
- Claude Desktop 설정은 `mcpServers`에 `command: "obscura", args: ["mcp"]`만 넣으면 된다.

## 통합

- **Hermes 에이전트 플러그인**(`SGavrl/hermes-plugin-obscura`): NousResearch의 Hermes 에이전트 브라우저 작업을 Obscura에서 실행. 세션마다 `obscura serve`를 띄우거나 기존 서버에 연결해 CDP로 구동하며 `--stealth` 옵션도 지원.

## 관련 노트

- [[github-scraping-repos]] — 스크래핑용 오픈소스 저장소 10선
- [[firecrawl-open-lovable]] — Firecrawl 기반 웹 복제/스크래핑
- [[proxifly-free-proxy-list]] — 무료 로테이팅 프록시 리스트
- [[테스트 데이터 수집할 웹 사이트 선정]] — 스크래핑 대상 사이트 정리

> 원문: [h4ckf0r0day/obscura: The headless browser for AI agents and web scraping](https://github.com/h4ckf0r0day/obscura)
> 원본 클립: [[2026-07-14-h4ckf0r0dayobscura The headless browser for AI agents and web scraping]]
