---
title: "proxifly/free-proxy-list — 100+개국 HTTPS·SOCKS5 무료 로테이팅 프록시 리스트"
date: 2026-07-14 14:32:15 +0900
categories: [개발, 도구]
tags: [proxy, socks5, https, api, npm, opensource, scraping]
slug: proxifly-free-proxy-list
publish: true
---
**Proxifly**는 5분마다 웹 곳곳에서 HTTP·HTTPS·SOCKS4·SOCKS5 프록시를 새로 수집·검증해 무료로 제공하는 프로젝트다. 최신 업데이트(2026-07-14 04:11 UTC) 기준 **111개국에서 6,067개**의 동작하는 프록시를 확보했다.

## 특징

- 매우 빠르고, **5분마다 검증**된다
- **HTTP / HTTPS / SOCKS4 / SOCKS5** 프로토콜별로 분류
- 111개국의 프록시 포함, 중복 없음
- **.json / .txt / .csv** 세 가지 포맷 제공
- HTTPS 연결 지원

## 사용 방법

프록시를 받는 경로는 여러 가지다: 공식 웹사이트의 무료 프록시 리스트, 무료 프록시 스크레이퍼 소프트웨어, 직접 다운로드 링크, NPM 모듈, cURL.

이용 시 [GitHub Acceptable Use Policy](https://docs.github.com/en/site-policy/acceptable-use-policies/github-acceptable-use-policies)를 따르고, 남용하거나 불법 행위에 쓰지 말라는 안내가 붙어 있다.

### 직접 다운로드 (jsDelivr CDN)

최신 업데이트 기준 프로토콜별 개수와 링크 경로:

| 종류 | 개수 | 경로 |
| --- | --- | --- |
| 전체 | 6067 | `proxies/all/data.{json,txt,csv}` |
| HTTP | 592 | `proxies/protocols/http/data.*` |
| HTTPS | 1342 | `proxies/protocols/https/data.*` |
| SOCKS4 | 342 | `proxies/protocols/socks4/data.*` |
| SOCKS5 | 3791 | `proxies/protocols/socks5/data.*` |
| 미국 프록시 | 1302 | `proxies/countries/US/data.*` |

기본 URL은 `https://cdn.jsdelivr.net/gh/proxifly/free-proxy-list@main/` 이고, [국가별 프록시](https://github.com/proxifly/free-proxy-list/tree/main/proxies/countries) 디렉터리도 있다.

### NPM 모듈

공식 **Proxifly NPM 모듈**로 애플리케이션에서 최신 프록시를 쉽게 가져올 수 있다.

```
npm install proxifly
```

```javascript
// ESM
import Proxifly from 'proxifly';
// CommonJS
const Proxifly = require('proxifly');

const proxifly = new Proxifly({
  // 필수는 아니지만, API 키가 있으면 제한이 풀린다 (https://proxifly.dev 에서 발급).
  apiKey: 'your-api-key'
});

proxifly.getProxy({
  protocol: 'http',      // http | socks4 | socks5
  anonymity: 'elite',    // transparent | anonymous | elite
  country: 'US',         // ISO 국가 코드
  https: true,
  format: 'json',        // json | text
  quantity: 1,           // 1 - 20
})
.then(proxy => console.log('Proxies:', proxy))
.catch(e => console.error(e));
```

### cURL로 가져오기

```
curl -sL https://cdn.jsdelivr.net/gh/proxifly/free-proxy-list@main/proxies/all/data.txt -o all.txt
curl -sL https://cdn.jsdelivr.net/gh/proxifly/free-proxy-list@main/proxies/protocols/http/data.txt -o http.txt
curl -sL https://cdn.jsdelivr.net/gh/proxifly/free-proxy-list@main/proxies/protocols/socks4/data.txt -o socks4.txt
curl -sL https://cdn.jsdelivr.net/gh/proxifly/free-proxy-list@main/proxies/protocols/socks5/data.txt -o socks5.txt
curl -sL https://cdn.jsdelivr.net/gh/proxifly/free-proxy-list@main/proxies/countries/US/data.txt -o socks5.txt
```

## 기여

기여는 언제나 환영이며, 작은 도움도 크게 감사받고 크레딧이 항상 주어진다. 관련 링크: [Site](https://proxifly.dev/) · [NPM Module](https://www.npmjs.com/package/proxifly) · [GitHub Repo](https://github.com/proxifly/free-proxy-list)

## 관련 노트

- [[obscura-headless-browser]] — AI 에이전트·스크래핑용 헤드리스 브라우저, HTTP/SOCKS5 프록시 지원
- [[github-scraping-repos]] — 인터넷 스크래핑용 오픈소스 저장소 10선(프록시 로테이션 포함)
- [[firecrawl-open-lovable]] — 스크래핑 기반으로 웹을 React 앱으로 복제
- [[테스트 데이터 수집할 웹 사이트 선정]] — 스크래핑 대상 사이트 후보 정리

> 원문: [proxifly/free-proxy-list: a free rotating proxy API with tested HTTPS and SOCKS5 proxies from 100+ countries](https://github.com/proxifly/free-proxy-list)
> 원본 클립: [[2026-07-14-proxiflyfree-proxy-list a free rotating proxy API with tested HTTPS and SOCKS5 proxies from 100+ countries]]
