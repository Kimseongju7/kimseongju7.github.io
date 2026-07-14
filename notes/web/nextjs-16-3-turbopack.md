---
title: "Turbopack: Next.js 16.3의 새로운 기능"
date: 2026-07-14 14:30:42 +0900
categories: [개발, 프론트엔드]
tags: [nextjs, turbopack, react, bundler, hmr, performance, react-compiler]
slug: nextjs-16-3-turbopack
publish: true
---
Next.js 16.3 프리뷰의 Turbopack 업데이트 정리. 이번 안정 릴리스는 컴파일러 성능에 집중해 dev 서버 메모리를 최대 90% 줄이고, 빌드용 영속 파일시스템 캐시와 실험적 Rust React Compiler, `import.meta.glob` API를 도입했다.

## 핵심 변경점 요약

- dev 서버 메모리 사용량 최대 90% 감소
- 더 빠른 빌드를 위한 영속 파일시스템 캐시
- 실험적 Rust React Compiler 지원
- `import.meta.glob` API 지원
- 더 빠른 HMR과 dev 시작 시간

## dev 모드 메모리 사용량 감소

Turbopack의 핵심 설계는 증분 컴파일(incremental compilation)이다. 이전에 수행한 작업을 캐싱해 변경되지 않은 파일의 재컴파일을 피하므로, 대형 앱에서 컴파일 시간은 라우트 크기가 아니라 변경 크기에 비례한다. 다만 이 설계는 "CPU를 아끼는 대신 메모리에 더 많이 캐싱"하는 의도적 트레이드오프였고, 코딩 에이전트·IDE·타입체커·린터가 함께 도는 요즘 개발 환경에서 메모리 압박의 원인이 됐다. 지난 3개월간 팀이 이를 집중 개선해, **16.3으로 올리면 장시간 dev 세션에서 즉시 메모리 감소를 체감할 수 있다.**

50개 라우트 컴파일 후 메모리 사용량:

- vercel.com (대시보드): 21.5 GB → 2 GB (~90% 감소)
- nextjs.org: 4,600 MB → 840 MB (~82% 감소)

상당 부분은 자료구조 압축, 불필요하게 오래 데이터를 들고 있지 않기 같은 작은 개선의 누적이지만, 가장 큰 성과는 **인메모리 캐시 축출(eviction)** 능력이다. Next.js 16.1에서 도입된 파일시스템 영속화를 활용해 캐시 결과를 메모리에서 내릴 수 있게 되면서, 방문한 모든 라우트를 메모리에 계속 붙들어 두는 무한 증가를 막는다.

- 메모리 축출은 개발용 파일시스템 캐시가 켜져 있어야 하며, 16.3에서는 둘 다 기본 활성화다.
- 캐시·성능 조사가 필요할 때는 실험 옵션으로 끌 수 있다: `experimental.turbopackMemoryEviction: false` (기본값은 `'full'`)
- 감소율은 앱마다 다르다 — 라우트 그래프 크기, 세션 중 건드린 범위, 세션 길이에 따라 달라진다.

## 빌드용 파일시스템 캐시

Turbopack 메모리 캐시의 디스크 영속화는 16.1부터 `next dev`를 빨라지게 해 왔는데, Vercel 자사 사이트에서 수개월 검증을 거쳐 이제 같은 영속 캐시를 `next build`에서도 쓸 수 있다.

`next build` 컴파일 시간(콜드 → 캐시):

- nextjs.org: 21s → 9.2s (~2.3배 빠름)
- vercel.com/home: 66s → 46s (~1.4배 빠름)
- vercel.com/geist: 30s → 5.5s (~5.5배 빠름)

CI에서는 생성된 `.next` 디렉터리를 다음 실행으로 복사해 활용하면 된다. 빌드 시작 시 캐시가 보이면 Turbopack이 새 변경을 컴파일하기 전에 디스크에서 항목을 읽는다. 활성화 플래그는 `experimental.turbopackFileSystemCacheForBuild: true`.

## 실험적 Rust React Compiler

React Compiler는 16.0부터 안정 지원됐지만 지금까지는 Babel 트랜스폼으로만 제공돼, 큰 앱에서 JS 실행 리소스를 기다리느라 빌드가 느려질 수 있었다. 최근 React 팀이 컴파일러의 네이티브 Rust 포트를 공개했고, Turbopack이 이를 빠르게 통합했다. [v0](https://v0.app/) 같은 대형 React 앱 대상 초기 테스트에서 컴파일 20~50% 개선을 보여, 실험 기능으로 공개한다.

```
const nextConfig = {
  reactCompiler: true,
  experimental: {
    turbopackRustReactCompiler: true, // Babel 버전 대신 Rust 네이티브 사용
  }
}
```

## import.meta.glob

Turbopack이 Vite 호환 `import.meta.glob` API를 지원한다.

```
const posts = import.meta.glob('./posts/*.mdx');
```

- 패턴에 맞는 모든 모듈을 이름 하드코딩 없이 가져오며, 결과는 매칭된 파일 경로를 키로 하는 객체다.
- 기본적으로 각 값은 모듈을 로드하는 async 함수이고, `eager: true`를 주면 각 매치를 즉시 `import`한다.
- named import, 다중 패턴, 부정 패턴, 커스텀 검색 경로, 로더용 쿼리 스트링, TypeScript 타입 생성도 지원한다.
- Turbopack의 파일 워처 기반이라 매치 집합에 파일이 추가/삭제되면 dev 모드에서 재컴파일이 트리거된다. 블로그 글이나 상품 설명처럼 유사 문서 집합을 불러올 때 알맞다.
- Turbopack 전용 기능으로, `--webpack` 옵션으로 빌드한 Next.js 앱에서는 동작하지 않는다.

## HMR 개선과 런타임 축소

- Vercel의 대형 Next.js 앱에서 성능을 분석해 HMR 구독을 더 효율적으로 만들었다. 페이지에 로드된 청크 추적을 여러 구독에서 단일 구독으로 합치는 변경으로, 복잡한 앱의 dev 서버 [[콜드 스타트]]를 15% 이상 단축했다. 향후 릴리스에서 추가 메모리·콜드 스타트 개선이 예고돼 있다.
- **런타임 크기 축소**: Turbopack은 모듈 해석과 청크 동적 로딩용 런타임 코드를 모든 라우트에 실어 보내는데, WebAssembly·워커·톱레벨 async 모듈 로딩 코드는 이제 실제로 필요할 때만 포함한다.

## 로컬 PostCSS 설정

모노레포에서는 패키지마다 다른 PostCSS 변환이 필요할 수 있다. 실험 옵션 `experimental.turbopackLocalPostcssConfig: true`를 켜면 Turbopack이 각 CSS 파일에서 가장 가까운 설정을 먼저 찾고, 없으면 프로젝트 루트로 폴백한다. 패키지 수준 CSS는 로컬 설정을, 애플리케이션 CSS는 루트 설정을 쓸 수 있다.

## 호환성과 안정성

16.3은 16.2 패치 라인의 모든 수정을 포함하며 모듈 해석·트레이싱·HMR 전반을 개선했다.

- Windows에서 `import.meta.url`의 올바른 파일 URL
- 청크 페치 실패 시 재시도
- `createRequire(new URL(..., import.meta.url))` 지원 개선
- `worker_threads` URL 해석 수정
- `module-sync` export 조건 지원
- webpack 로더 크래시 시 더 나은 에러 메시지
- Safari의 CSS HMR 수정

## 관련 노트

- [[js-framework-hype-cycle]] — Next.js가 지나온 JS 프레임워크 유행사 속에서의 현재 위치
- [[메타프레임워크]] — Next.js를 포함한 메타프레임워크 개념 정리
- [[JS번들]] — Turbopack이 대체하는 자바스크립트 번들러 개념

> 원문: [Turbopack: What's New in Next.js 16.3](https://nextjs.org/blog/next-16-3-turbopack)
> 원본 클립: [[2026-07-14-Turbopack What's New in Next.js 16.3]]
