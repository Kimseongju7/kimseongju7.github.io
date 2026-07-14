---
title: "firecrawl/open-lovable — 어떤 웹사이트든 몇 초 만에 모던 React 앱으로 복제"
date: 2026-07-14 14:32:46 +0900
categories: [개발, 도구]
tags: [firecrawl, react, ai, opensource, nextjs, sandbox, llm]
slug: firecrawl-open-lovable
publish: true
---
**Open Lovable**은 AI와 채팅하며 React 앱을 즉시 만들어 주는 오픈소스 예제 앱이다. [Firecrawl](https://firecrawl.dev/?ref=open-lovable-github) 팀이 만들었으며, 어떤 웹사이트든 몇 초 만에 모던 React 앱으로 복제·재현하는 것을 내세운다. 완전한 클라우드 솔루션이 필요하면 [Lovable.dev](https://lovable.dev/)를 참고하라고 안내한다. 라이선스는 MIT.

## 설치와 실행

1. 클론 후 의존성 설치:

```
git clone https://github.com/firecrawl/open-lovable.git
cd open-lovable
pnpm install  # 또는 npm install / yarn install
```

2. `.env.local` 작성 (아래 환경 변수 참고)
3. 실행:

```
pnpm dev  # 또는 npm run dev / yarn dev
```

이후 http://localhost:3000 접속.

## 환경 변수 구성

`.env.local`은 네 부분으로 나뉜다.

- **필수**: `FIRECRAWL_API_KEY` — https://firecrawl.dev 에서 발급.
- **AI 프로바이더** (사용할 LLM 선택): `GEMINI_API_KEY`, `ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, `GROQ_API_KEY` 중 선택.
- **Fast Apply** (선택, 더 빠른 편집용): `MORPH_API_KEY` — morphllm.com 대시보드에서 발급.
- **샌드박스 프로바이더**: `SANDBOX_PROVIDER=vercel`(기본) 또는 `e2b` 중 하나 선택.

## 샌드박스 인증 방식

**옵션 1 — Vercel Sandbox (기본)**, 인증 방법 두 가지 중 택일:

- 방법 A: OIDC 토큰 (개발 환경 권장) — `vercel link` 후 `vercel env pull`을 실행하면 `VERCEL_OIDC_TOKEN`이 자동 생성된다.
- 방법 B: 개인 액세스 토큰 (프로덕션 또는 OIDC 불가 시) — `VERCEL_TEAM_ID`, `VERCEL_PROJECT_ID`, `VERCEL_TOKEN`(Vercel 대시보드에서 발급)을 설정한다.

**옵션 2 — E2B Sandbox**: `E2B_API_KEY` — https://e2b.dev 에서 발급.

## 관련 노트

- [[github-scraping-repos]] — Firecrawl이 소개된 스크래핑 저장소 10선
- [[obscura-headless-browser]] — 스크래핑용 헤드리스 브라우저
- [[proxifly-free-proxy-list]] — 무료 로테이팅 프록시 리스트

> 원문: [firecrawl/open-lovable: 🔥 Clone and recreate any website as a modern React app in seconds](https://github.com/firecrawl/open-lovable)
> 원본 클립: [[2026-07-14-firecrawlopen-lovable 🔥 Clone and recreate any website as a modern React app in seconds]]
