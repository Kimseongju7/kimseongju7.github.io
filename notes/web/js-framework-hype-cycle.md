---
title: "프레임워크 호핑을 멈춰라 — JS 프레임워크 유행 10년사"
date: 2026-07-14 14:33:05 +0900
categories: [개발, 아티클]
tags: [javascript, frontend, framework, react, nextjs, fundamentals]
slug: js-framework-hype-cycle
publish: true
---
2017년부터 2026년까지 매년 "이 프레임워크가 미래다"가 반복되어 온 JS 생태계의 유행 사이클을 풍자하며, 프레임워크를 쫓아다니지 말고 웹의 기본기를 먼저 배우라는 트윗이다.

## 10년간의 프레임워크 유행 연대기

- 2017: "React가 미래다!"
- 2018: "React는 별로야, Vue.js를 써봐"
- 2019: "React가 함수형으로 간다, Vue는 잊어"
- 2020: "Vue는 별로야, Next.js를 써봐!"
- 2021: "Next.js가 미래다!"
- 2022: "Next.js는 비대해, Remix를 써봐"
- 2023: "Remix는 너무 초기야, Astro를 써봐"
- 2024: "Astro는 한계가 있어, Next.js가 괜찮았지"
- 2025: "Next.js는 벤더 락인이야, TanStack을 써"
- 2026: "TanStack이 미래다..."

## 핵심 주장: 도구가 아니라 프로세스를 이해하라

- "프레임워크 호핑(framework hopping)"을 멈춰라. 새로운 트렌드를 계속 쫓다 보면 성장하지 못한다.
- 웹의 기본을 먼저 배워라:
  - JavaScript, TypeScript, [[DOM]], [[CSR]], [[SSR]], ISG 등
  - 브라우저가 UI를 어떻게 렌더링하는지
  - LCP, FCP, 렌더 캐싱
- 더 나은 웹 개발자가 되려면 도구가 아니라 프로세스를 이해해야 한다. 도구는 변하지만 프로세스는 그대로이기 때문이다.
- 진짜 본질을 배우기에 지금보다 좋은 시기는 없다.

## 반응 (주요 댓글)

- "몇 년마다 도구는 바뀌지만 HTTP, HTML, CSS, JavaScript는 여전히 그대로다."
- "브라우저는 똑같고, 오직 래퍼(wrapper)만 바뀐다." (작성자)
- "유일한 상수는 '내년에 모든 걸 마침내 고친다'는 프레임워크다." → 작성자: "그럴 일 없다. 그랬다간 다음 프레임워크가 해결해야 할 새로운 문제를 만들어낼 것."
- 여러 댓글이 "반짝이는 프레임워크를 쫓느라 몇 년을 낭비했고, 이제 기본기에 집중한다"고 공감했다.

## 관련 노트

- [[nextjs-16-3-turbopack]] — 연대기 속 Next.js의 최신 버전에서 실제로 무엇이 바뀌는지
- [[메타프레임워크]] — Next.js·Remix·Astro·TanStack을 아우르는 메타프레임워크 개념
- [[Frontend 공부하기]] — "도구가 아니라 기본기부터"라는 조언과 이어지는 프론트엔드 학습 노트

> 원문: [avrl ☘ — "2017: React is the future!" ... framework hopping thread](https://x.com/avrldotdev/status/2070763048512672146)
> 원본 클립: [[2026-07-14-X에서 avrl ☘ 님  "2017 "React is the future!"2018 "React isn't good, try Vue.js"2019 "React goes functional, forget Vue"2020 "Vue isn't good, try Next.js!"2021 "Next.js is the future!"2022 "Next.js is bloated, try Remix"2023]]
