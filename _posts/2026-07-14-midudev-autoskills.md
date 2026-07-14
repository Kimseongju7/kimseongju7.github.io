---
title: 'midudev/autoskills — 명령 하나로 AI 스킬 스택 전체 설치'
date: 2026-07-14 14:06:53 +0900
categories: [개발, 도구]
tags: [skills, ai-agent, cli, npx, tooling, security, open-source]
description: 'autoskills는 npx autoskills 명령 하나로 프로젝트를 스캔해 기술 스택을 감지하고, 거기에 맞는 큐레이션된 AI 에이전트 스킬을 자동 설치해 주는 도구다. 설정이 전혀 필요 없다.'
---
autoskills는 `npx autoskills` 명령 하나로 프로젝트를 스캔해 기술 스택을 감지하고, 거기에 맞는 큐레이션된 AI 에이전트 스킬을 자동 설치해 주는 도구다. 설정이 전혀 필요 없다.

## 동작 방식

1. 프로젝트 루트에서 `npx autoskills` 실행
2. `package.json`, Gradle 파일, 설정 파일을 스캔해 사용 기술을 감지
3. 감사(audit)된 autoskills 레지스트리에서 가장 잘 맞는 AI 에이전트 스킬을 선택
4. 선택된 스킬 파일만 레지스트리에서 내려받아 검증한 뒤 로컬에 기록

## 보안 모델

- 런타임에 임의의 업스트림 저장소에서 직접 설치하지 않는다.
- 스킬은 메인테이너가 저장소 내 autoskills 레지스트리로 동기화하고, 프롬프트 인젝션·공급망 리스크를 스캔한 뒤 SHA-256 해시를 매니페스트에 기록한다.
- CLI는 프로젝트에 필요한 스킬만 큐레이션된 레지스트리에서 내려받아 모든 파일을 매니페스트와 대조 검증하고, 설치 소스와 번들 해시를 담은 `skills-lock.json` 항목을 작성한다.
- 이 방식으로 패키지를 작게 유지하면서 설치 중 서드파티 스킬 소스에서의 라이브 다운로드를 피한다.

## 옵션

```
-y, --yes       확인 프롬프트 생략
--dry-run       설치하지 않고 설치될 내용만 표시
-h, --help      도움말 표시
```

## 지원 기술

모던 프론트엔드·백엔드·모바일·클라우드·미디어 스택 전반을 지원한다.

- **프레임워크 & UI**: React, Next.js, Vue, Nuxt, Svelte, Angular, Astro, Tailwind CSS, shadcn/ui, GSAP, Three.js
- **언어 & 런타임**: TypeScript, Node.js, Go, Bun, Deno, Dart
- **백엔드 & API**: Express, Hono, NestJS, Spring Boot
- **모바일 & 데스크톱**: Expo, React Native, Flutter, SwiftUI, Android, Kotlin Multiplatform, Tauri, Electron
- **데이터 & 스토리지**: Supabase, Neon, Prisma, Drizzle ORM, Zod, React Hook Form
- **인증 & 결제**: Better Auth, Clerk, Stripe
- **테스팅**: Vitest, Playwright
- **클라우드 & 인프라**: Vercel, Vercel AI SDK, Cloudflare, Durable Objects, Cloudflare Agents, Cloudflare AI, AWS, Azure, Terraform
- **툴링**: Turborepo, Vite, oxlint
- **미디어 & AI**: Remotion, ElevenLabs

## 관련 노트

- [mattpocock-skills](/posts/mattpocock-skills/) — 작고 조합 가능한 엔지니어링 스킬 모음
- [codex-top-6-skills](/posts/codex-top-6-skills/) — Codex 에이전트를 확장하는 대표 스킬 6선
- [alirezarezvani-claude-skills](/posts/alirezarezvani-claude-skills/) — Claude용 스킬 모음
- [archify-diagram-skill](/posts/archify-diagram-skill/) — 채팅으로 아키텍처 다이어그램을 그리는 에이전트 스킬

> 원문: [midudev/autoskills: One command. Your entire AI skill stack. Installed.](https://github.com/midudev/autoskills)
> 원본 클립: 2026-07-14-midudevautoskills One command. Your entire AI skill stack. Installed.
