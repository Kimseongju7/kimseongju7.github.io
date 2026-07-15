---
title: kramdown
date: 2026-07-01 00:19:22 +0900
categories:
  - 학습
  - 용어
tags:
  - kramdown
  - markdown
  - jekyll
  - ruby
  - static-site
publish: true
---
## 한 줄 정의
kramdown은 루비(Ruby)로 만들어진 마크다운 파서·변환기로, Jekyll이 기본으로 쓰는 "마크다운 → HTML" 엔진이다.

## 자세히
**kramdown(크램다운)** 은 마크다운 문법으로 쓴 글을 HTML로 바꿔 주는 변환기다. 이름은 "**k**ramdown is a Markdown-superset converter"에서 왔는데, 표준 마크다운보다 기능이 더 많은 **상위 호환(superset)** 이라는 뜻이다. 순수 루비로 구현돼 있어 빠르고, 문법이 잘 정리돼 있어 Jekyll 같은 정적 사이트 생성기에서 표준 마크다운 처리기로 채택했다.

Jekyll에서 우리가 `notes/`에 쓰는 마크다운이 실제 HTML 페이지가 되는 그 변환 단계를 담당하는 게 바로 kramdown이다. 즉 [[Jekyll·Chirpy|Jekyll]]이 "글→사이트" 빌드를 총괄하는 엔진이라면, 그 안에서 본문 마크다운 한 덩어리를 `<h2>`, `<p>`, `<code>` 같은 HTML 태그로 바꿔 주는 실무 일꾼이 kramdown이다. `_config.yml`에서 `markdown: kramdown`으로 지정돼 있고, Chirpy도 이 기본값을 그대로 쓴다.

kramdown이 표준 마크다운에 더해 주는 대표 기능들:
- **자동 헤더 ID(앵커)** — `## 제목`을 HTML로 바꿀 때 `<h2 id="제목">`처럼 ID를 자동으로 붙인다. 목차(TOC)와 `#제목` 형태의 내부 링크가 이 ID에 기댄다.
- **IAL(Inline Attribute List)** — `{: .class #id }` 문법으로 특정 요소에 CSS 클래스·ID·속성을 직접 달 수 있다.
- **각주, 정의 목록, 표(table)** 등 표준 마크다운에 없는 확장 문법.

이 자동 헤더 ID가 한글에서 말썽을 부린다. kramdown은 헤더 텍스트로 ID를 만드는데, 비ASCII(한글 등) 문자 처리 방식이나 빌드 환경에 따라 ID가 깨지거나 비어서, 본문 안에서 `[가기](#소제목)`처럼 건 **내부 앵커 링크가 실제 ID와 어긋나는** 일이 생긴다. 그러면 빌드 후 htmlproofer 같은 링크 검사 도구가 "가리키는 앵커가 없다"며 빌드를 실패시킨다. (이 볼트에서 *"깨진 한글 헤더 내부 앵커 링크 평문화"* 커밋으로 해당 링크들을 평문으로 풀어 빌드 실패를 고친 적이 있는데, 그 근본 원인이 바로 kramdown의 헤더 ID 생성 동작이다.)

## 예시
`_config.yml`에서 마크다운 엔진을 지정하는 부분:

```yaml
markdown: kramdown
kramdown:
  input: GFM            # GitHub Flavored Markdown 문법 허용
  syntax_highlighter: rouge
```

같은 마크다운이 kramdown을 거치면 이렇게 변환된다:

```markdown
## 설치 방법
```

```html
<h2 id="설치-방법">설치 방법</h2>
```

이렇게 자동으로 붙은 `id`가 있어야 `[설치 방법으로](#설치-방법)` 같은 내부 링크나 목차가 동작한다. 한글 헤더라면 이 ID가 환경에 따라 달라질 수 있으니, 내부 앵커 링크를 걸 때는 빌드 후 실제 생성된 ID를 확인하는 게 안전하다.

## 관련
- [[Jekyll·Chirpy]]
- [[슬러그]]
- [공식 문서](https://kramdown.gettalong.org/)
