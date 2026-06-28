---
title: Jekyll·Chirpy
date: 2026-06-28 22:59:13 +0900
categories:
  - 학습
  - 용어
tags:
  - jekyll
  - chirpy
  - github-pages
  - static-site
  - blog
publish: true
---
## 한 줄 정의
Jekyll은 마크다운 글을 정적 웹사이트(HTML)로 변환해 주는 정적 사이트 생성기이고, Chirpy는 그 Jekyll 위에서 동작하는 블로그용 테마다.

## 자세히
**Jekyll(지킬)** 은 루비(Ruby)로 만들어진 정적 사이트 생성기(SSG, Static Site Generator)다. 미리 써 둔 마크다운 글과 레이아웃 템플릿을 받아서, 방문자가 보는 완성된 HTML 페이지들을 한 번에 빌드해 둔다. 워드프레스처럼 매 요청마다 서버에서 DB를 조회해 페이지를 그리는 게 아니라, 빌드 시점에 모든 페이지를 미리 만들어 두기 때문에 빠르고, 서버 없이 정적 파일만 호스팅하면 된다. 그래서 **GitHub Pages** 와 궁합이 좋다 — GitHub Pages가 Jekyll을 기본 지원하므로, 글을 깃에 push 하면 GitHub Actions가 Jekyll 빌드를 돌려 사이트를 자동 배포한다(루비를 로컬에 깔지 않아도 된다).

글의 메타데이터(제목·날짜·카테고리·태그 등)는 파일 맨 위 `---` 사이의 **front matter** 라는 YAML 블록에 적는다. Jekyll은 이 front matter와 본문 마크다운을 읽어, 정해진 레이아웃에 끼워 넣어 페이지를 만든다.

**Chirpy(처피)** 는 그 Jekyll에서 쓰는 **테마**다. 테마는 사이트의 디자인·레이아웃·기능을 묶어 둔 패키지라고 보면 된다. Chirpy는 개발 기술 블로그에 맞춰져 있어서 다크모드, 사이드바 카테고리/태그 메뉴, 목차(TOC), 검색, 코드 하이라이트 같은 기능을 기본 제공한다. 즉 **Jekyll = 엔진(글→사이트 변환), Chirpy = 그 위에 입히는 외관·기능** 의 관계다.

Chirpy를 쓸 때 알아둘 제약 두 가지: 카테고리는 **최대 2단계**(예: `[학습, 용어]`)까지만 의미 있게 처리되고, 태그는 **소문자**로 통일하는 게 좋다. 이 볼트의 글들이 front matter에서 그 규칙을 따르고 있는 이유다.

## 예시
이 블로그(`kimseongju7.github.io`)도 Jekyll + Chirpy로 돌아간다. 글의 front matter는 이런 형태다:

```yaml
---
title: 블로그 시작
date: 2026-06-28
categories: [일지]      # Chirpy는 2단계까지
tags: [chirpy, jekyll]  # 소문자 권장
slug: hello-blog        # URL용 이름
publish: true
---
```

워크플로 요약:

1. 옵시디언 `notes/` 에 마크다운으로 글을 쓴다.
2. front matter에 `title / date / categories / tags` 등을 채운다.
3. 깃에 commit & push → GitHub Actions가 Jekyll(+Chirpy)로 빌드 → GitHub Pages에 배포.

## 관련
- [[블로그 시작]]
- [[슬러그]]
