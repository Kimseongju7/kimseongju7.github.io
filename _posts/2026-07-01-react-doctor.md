---
title: 'React Doctor'
date: 2026-07-01 01:05:00 +0900
categories: [학습, 용어]
tags: [react, ai, ai-agent, linter, oxlint, 코드품질, frontend]
description: 'AI가 만든 React 코드의 나쁜 패턴을 규칙 기반으로 잡아내는 오픈소스 코드 스캐너 React Doctor를 정리한다'
---
## 한 줄 정의
AI 코딩 에이전트나 개발자가 만들어내는 **잘못된 React 패턴을 규칙 기반(결정론적)으로 잡아내는 코드 스캐너** — 코드 건강검진 도구.

## 자세히
표어가 핵심을 요약한다: *"Your agent writes bad React. This catches it."* (네 AI가 나쁜 React 코드를 짠다, 이게 그걸 잡아낸다.) 요즘은 코드를 직접 짜지 않고 AI(Claude Code, Cursor 등)에게 시키는 경우가 많은데, AI가 만든 코드는 **동작은 하지만 품질이 나쁜 경우**가 많다 — 불필요한 리렌더, 메모리 누수, 보안 구멍 등. React Doctor는 이런 "겉보기엔 멀쩡한데 사실 나쁜 코드"를 자동으로 찾아준다. Million.js 팀이 만들었고 MIT 라이선스 오픈소스다.

작동 원리의 핵심은 **"결정론적(deterministic)"**이라는 점. AI가 코드를 짜는 건 "찍어서 답 쓰기"(확률적)에 가깝다면, React Doctor는 "정해진 채점 기준표로 채점"(규칙대로, 항상 같은 결과)하는 것이다. 내부에 *"이런 패턴은 나쁘다"*는 규칙 체크리스트가 들어있고, 코드를 읽으며 거기에 걸리는 부분을 기계적으로 대조한다. 사람의 코드 리뷰를 자동·기계적으로 해주는 셈.

### 기술 스택 / 아키텍처
- **언어**: TypeScript (코드베이스 99%)
- **빌드**: Vite + Turbo(모노레포 빌드 캐싱)
- **패키지 관리**: pnpm workspace (모노레포)
- **분석 엔진**: **Oxlint**(Rust로 작성된 초고속 린터) 기반 러너 + 자체 check 파이프라인
- **배포 형태**: npm 패키지(`npx react-doctor`) + GitHub Action
- **텔레메트리**: Sentry (익명 사용량, `--no-telemetry`로 옵트아웃 / 파일 내용·구체적 발견은 안 보냄)

단일 코어(`core`) 위에 CLI·CI·LSP(language-server)·에디터 확장(VSCode/Zed)·린터 플러그인(ESLint/Oxlint)을 모노레포로 함께 제공한다. Oxlint(Rust)를 써서 ESLint 대비 매우 빠른 게 특징.

### 검사 카테고리
- **State & Effects** — 잘못된 `useEffect`/상태 관리 패턴
- **Performance** — [불필요한 리렌더](/posts/virtual-dom/) 등 성능 안티패턴
- **Architecture** — 구조적 문제
- **Security** — 보안 스캔, 공급망(supply-chain), pnpm hardening
- **Accessibility** — 접근성(reduced-motion 등)
- **프레임워크 자동 감지** — Next.js, Vite, React Native, Expo, React Server Components

## 예시
사용법 — 터미널에 명령어 한 줄 (설치 불필요):

```bash
# 1) 그냥 한 번 검사 (기본)
npx react-doctor@latest

# 2) AI한테 미리 가르쳐두기 (예방)
#    Claude Code, Cursor, Codex, OpenCode 지원
npx react-doctor@latest install

# 3) CI에 붙이기 (PR마다 자동 검사, "새로 생긴 문제만" 코멘트)
npx react-doctor@latest ci install

# 텔레메트리 끄기
npx react-doctor@latest --no-telemetry
```

규칙·게이트·스캔 범위는 `doctor.config.ts`로 커스터마이징. **두 박자 방어**가 핵심 — ① AI가 코드 짤 때 나쁜 패턴 안 만들게 예방(`install`), ② 그래도 새는 건 검사로 잡아냄(`audit`/`ci`).

## 관련
- [Loop Engineering](/posts/loop-engineering/) — "AI가 짠 코드를 사람 대신 검증한다"는 점에서 루프의 *Verification(검증) / 평가자(Evaluator)* 역할과 같은 문제의식
- [Virtual DOM](/posts/virtual-dom/) — Performance 카테고리가 잡아내는 불필요한 리렌더는 가상 DOM 재조정 비용과 직결된 문제
- [컴포넌트](/posts/component/) — React 컴포넌트의 상태·이펙트·구조 패턴을 검사 대상으로 삼는 스캐너
- GitHub: millionco/react-doctor (★13.3k, MIT)
