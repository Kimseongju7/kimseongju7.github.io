---
title: "인터넷 전체를 스크래핑하기 위한 GitHub 저장소 10선"
date: 2026-07-14 14:32:26 +0900
categories: [개발, 도구]
tags: [scraping, crawler, github, open-source, firecrawl, scrapy, browser-automation]
slug: github-scraping-repos
publish: true
---
X의 Alejo(@ecommartinez)가 정리한 스레드로, 어떤 웹에서든 깨끗한 데이터를 추출할 수 있는 GitHub 오픈소스 저장소 10개를 소개한다. 이런 수준의 접근은 보통 영업 전화와 계약이 필요한 유료 서비스 영역이라는 것이 스레드의 요지다.

## 10개의 저장소

1. **[[firecrawl-open-lovable|Firecrawl]]** (`github.com/firecrawl/fire...`) — 어떤 웹사이트든 타겟팅해 모든 페이지를 추적하고 JavaScript를 렌더링하며, AI가 즉시 읽을 수 있는 깨끗하고 구조화된 데이터를 반환한다. 13만 스타로 GitHub 상위 100대 레포 중 하나. AI 스타트업의 절반이 조용히 쓰고 있는 스크래핑 도구.
2. **Crawl4AI** (`github.com/unclecode/craw...`) — GitHub 최고의 스크레이퍼로 소개. 어떤 웹도 LLM에 바로 쓸 수 있는 깨끗한 마크다운으로 변환하며 유료 서비스보다 빠르다. API 키·계정·페이지당 과금이 필요 없다. 한 개발자가 16달러짜리 유료 스크레이퍼에 질려 며칠 만에 만들었다. 5.1만 스타, Apache 2.0.
3. **browser-use** (`github.com/browser-use/br...`) — 사람처럼 브라우저를 다루는 AI 에이전트. 클릭·스크롤·로그인·양식 작성을 하고, 처음 보는 웹사이트에서도 데이터를 추출한다. 취리히 공과대학(ETH) 연구원 두 명이 개발했고 1년 만에 95,000 스타. 단순 스크레이퍼가 못 가는 곳까지 간다.
4. **Crawlee** (`github.com/apify/crawlee`) — 전문적이고 완전한 스크래핑 프레임워크. 프록시 로테이션, 자동 재시도, 브라우저 지문 위장, 큐 관리를 포함해 차단을 피하기 위한 모든 메커니즘을 제공한다. 스크래핑 회사들이 수천 달러를 청구하는 스택을 무료로 쓸 수 있다.
5. **Scrapy** (`github.com/scrapy/scrapy`) — 10년 넘게 데이터 팀들을 조용히 도와온 산업 수준의 스크레이퍼. 수백만 페이지를 추적하고 어떤 콘텐츠든 추출해 깔끔하게 내보낸다. 실제 조건에서 대규모로 검증됐고 항상 무료.
6. **MarkItDown** (`github.com/microsoft/mark...`) — 마이크로소프트의 자체 도구. PDF·Office 문서·HTML·이미지 등 모든 파일이나 웹을 AI가 쓸 수 있는 깔끔한 마크다운으로 변환한다. 이 도구를 중심으로 데이터 파이프라인을 구축한 기업들도 있다.
7. **Scrapling** (`github.com/D4Vinci/Scrapl...`) — 웹사이트 디자인 변경에 자동으로 적응하고 안티스크래핑 탐지를 피해가는 "보이지 않는" 스크레이퍼. 안티스크래핑 업체들이 프리미엄 기능으로 파는 기술을 무료 오픈소스로 제공한다.
8. **scrcpy** (`github.com/Genymobile/scr...`) — 컴퓨터에서 안드로이드 기기를 원격 제어해 데이터를 추출하거나 웹이 없는 앱을 자동화한다. 대부분의 스크레이퍼가 닿지 못하는 모바일 전용 플랫폼에 접근할 수 있다. 13만+ 스타, Apache 2.0.
9. **AutoScraper** (`github.com/alirezamika/au...`) — 예시 하나를 주면 패턴을 찾아 웹의 나머지를 자동으로 추적한다. 셀렉터 관리와 코드 유지보수가 필요 없는, 몇 줄의 Python 코드로 되는 "데이터 내놔" 버튼.
10. **curl-impersonate** (`github.com/lwthiker/curl-...`) — 실제 브라우저의 지문을 완벽하게 모방하는 curl의 개선판. 요청이 Chrome을 쓰는 실제 사람처럼 보인다. 비싼 스크래핑 API들이 저수준에서 비밀리에 쓰는 트릭이며, 이걸로 월 2,000달러를 청구하는 회사들도 있다.

## 메모

- 스레드 원문은 스페인어이며, 클립은 X의 한국어 자동 번역 화면을 담고 있다.
- 목록의 링크는 t.co 단축 링크라 저장소 전체 이름이 잘려 있다. 위 저장소 이름은 문맥(설명·스타 수·라이선스)으로 식별한 것이다.

## 관련 노트

- [[firecrawl-open-lovable]] — 목록의 Firecrawl로 웹을 React 앱으로 복제
- [[obscura-headless-browser]] — Rust제 헤드리스 브라우저(스크래핑·안티디텍트)
- [[proxifly-free-proxy-list]] — 차단 우회용 무료 프록시 리스트
- [[테스트 데이터 수집할 웹 사이트 선정]] — 스크래핑 대상 사이트 후보

> 원문: [X에서 Alejo 님: "10 repositorios de GitHub para scrapear todo internet…"](https://x.com/ecommartinez/status/2071360257906168143)
> 원본 클립: [[2026-07-14-X에서 Alejo 님  "10 repositorios de GitHub para scrapear todo internetGuárdalos todos. Cada uno extrae datos limpios de cualquier web. Ese nivel de acceso normalmente exige llamadas de ventas y contratos. httpst.coqw3BR19Qx2]]
