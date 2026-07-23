---
title: "OpenObserve — Datadog·Splunk·Elasticsearch를 대체하는 오픈소스 관측성 플랫폼"
date: 2026-07-14 14:25:22 +0900
categories: [개발, 도구]
tags: [observability, open-source, rust, logs, metrics, tracing, opentelemetry, datadog, elasticsearch]
slug: openobserve
publish: true
---
OpenObserve(O2)는 로그, 메트릭, 트레이스, 애널리틱스, RUM(Real User Monitoring)을 하나로 다루는 클라우드 네이티브 관측성(observability) 플랫폼이다. Datadog·Splunk·Elasticsearch의 비용 효율적 대안을 표방하며, 저장 비용 140배 절감과 단일 바이너리 배포가 핵심 셀링 포인트다.

## 왜 OpenObserve인가

| 장점 | 설명 |
| --- | --- |
| 140배 낮은 저장 비용 | Parquet 컬럼형 저장 + S3 네이티브 아키텍처로 Elasticsearch 대비 비용 대폭 절감 |
| 단일 바이너리 배포 | 2분 안에 구동, 복잡한 클러스터 설정 불필요 |
| OpenTelemetry 네이티브 | OpenTelemetry 표준 기반 — 벤더 종속 없음 |
| 통합 플랫폼 | 로그·메트릭·트레이스·RUM·대시보드·알림을 한 도구에 |
| 고성능 | 하드웨어 1/4로 Elasticsearch보다 나은 쿼리 성능 |
| SQL + PromQL | 로그/트레이스는 SQL, 메트릭은 SQL 또는 PromQL — 독자 쿼리 언어 없음 |
| Rust로 구현 | 메모리 안전, 고성능, 단일 바이너리 |

## 아키텍처

- **Parquet 컬럼형 저장**: 효율적인 압축과 쿼리 성능
- **S3 네이티브 설계**: 저렴한 오브젝트 스토리지 + 지능형 캐싱
- **파티셔닝·인덱싱·스마트 캐싱**: 대부분의 쿼리에서 검색 공간을 최대 99%까지 축소
- **네이티브 [[멀티테넌시]]**: 조직(organization)과 스트림(stream)이 일급 개념, 완전한 데이터 격리
- **스테이트리스 아키텍처**: 빠른 스케일링과 낮은 RPO/RTO(재해 복구)

규모와 배포:

- 수천 명의 동시 사용자가 단일 클러스터를 쿼리 가능
- 단일 바이너리로 테라바이트까지 확장 — 관측성 분야에서 유일
- HA([[고가용성]]) 모드는 페타바이트급 워크로드까지 확장
- 멀티 리전 배포(Super Cluster 클러스터 연합)와 연합 검색(federated search)은 엔터프라이즈 기능
- 스테이트리스 노드 + S3 기반 저장(11 nines 내구성)으로 매우 낮은 RPO/RTO

## 주요 기능

- **로그 관리**: 전문(full-text) 검색, SQL 쿼리, 강력한 필터링을 갖춘 중앙집중식 로그 관리. 로그 데이터로 대시보드를 만들고 알림 설정 가능.
- **분산 트레이싱**: OpenTelemetry 기반. Flamegraph와 Gantt 차트로 요청 분해를 보고, 스팬을 클릭해 전체 트레이스에서 병목을 파악.
- **메트릭·대시보드**: 19종 이상의 내장 차트 + 커스텀 차트로 200가지 이상의 시각화. SQL 또는 PromQL로 쿼리, 다중 쿼리를 수식으로 결합.
- **프론트엔드 모니터링(RUM)**: 성능 추적, 에러 로깅, 세션 리플레이로 사용자가 실제로 겪는 경험을 파악.
- **알림(Alerts)**: 모든 텔레메트리 신호(로그·메트릭·트레이스)에 알림·임계값·알림 채널 설정. 알림 히스토리와 이상 감지(anomaly detection) 지원.
- **파이프라인**: 수집 시점에 데이터 보강·마스킹·축소·정규화. 로그→메트릭 변환 같은 스트림 처리를 외부 도구 없이 수행.

## 시작하기

- **OpenObserve Cloud**: 인프라 관리 없이 수 분 내 시작, 무료 티어는 일 50GB 수집까지.
- **Docker**:

```
docker run -d \
      --name openobserve \
      -v $PWD/data:/data \
      -p 5080:5080 \
      -e ZO_ROOT_USER_EMAIL="root@example.com" \
      -e ZO_ROOT_USER_PASSWORD="Complexpass#123" \
      public.ecr.aws/zinclabs/openobserve:latest
```

프로덕션 검증: 전 세계 수천 개의 활성 배포, 최대 배포는 **일 2PB 이상 수집**.

## 기존 도구와의 비교

- **vs Datadog**: 셀프 호스트/클라우드 선택 가능(vs SaaS 전용), GB당 과금(일 200GB까지 무료) vs 호스트당+GB당, 오픈소스(AGPL-3.0), 네이티브 OTLP, SQL+PromQL vs 독자 언어, 벤더 종속 없음.
- **vs Elasticsearch**: 저장 비용 140배 절감(vs hot/warm/cold 티어), 단일 바이너리 vs 복잡한 클러스터 관리, SQL vs Lucene/KQL, 하드웨어 요구 1/4.
- **vs Splunk**: 오픈소스 vs 비싼 엔터프라이즈 라이선스, 단일 바이너리 또는 HA 클러스터 vs 복잡한 배포, SQL+PromQL vs SPL, 예측 가능한 낮은 비용.
- **vs Grafana/Loki/Prometheus 스택**: 단일 플랫폼 vs 여러 도구 조합(Grafana+Loki+Prometheus+Tempo), 바이너리 하나로 관리, 고카디널리티 완전 지원(Loki는 고카디널리티에 취약), 대용량에서 빠른 쿼리.

## 보안·컴플라이언스

- 보안 기능: 안전한 컨테이너 이미지, 민감 데이터 마스킹(SDR, 엔터프라이즈), 저장·전송 중 암호화, SSO(OIDC·OAuth·SAML·LDAP/AD, 엔터프라이즈), RBAC(엔터프라이즈).
- 인증: SOC 2 Type II, ISO 27001, GDPR 준수, HIPAA 대응(엔터프라이즈 계약 시 BAA 가능).

## 라이선스와 엔터프라이즈

- **오픈소스판**: AGPL-3.0. 개선 사항이 오픈소스로 남아 커뮤니티에 환원되도록 하기 위한 선택이며, 상업적 사용도 자유롭다. 오픈소스판은 기능 완결적(feature-complete)이고 프로덕션 레디다.
- **엔터프라이즈판**: 상용 라이선스. SSO, 고급 RBAC, 감사 추적(audit trail), Super Cluster 연합 검색, SDR, AES-256 SIV 고급 암호화, 쿼리 관리, 워크로드 QoS 등 추가. 무료 티어는 일 50GB 수집(월 약 1.5TB)까지이며 상업적 사용 포함.

## FAQ 요점

- **140배 절감의 원리**: Parquet 컬럼형 포맷(효율적 압축) + S3 네이티브 아키텍처(저렴한 오브젝트 스토리지)의 조합.
- **제약**: 모든 데이터는 **불변(immutable)** — 수집 후 수정·삭제 불가(보존 기간 전체 삭제만 가능). 로그·컴플라이언스 관점에서는 데이터 무결성과 감사 추적을 보장하는 의도된 설계.
- **학습 곡선**: SQL/PromQL 등 익숙한 쿼리 언어, 드래그 앤 드롭 대시보드 빌더, 샤드·레플리카·힙 크기 같은 복잡한 튜닝 불필요. 대부분 몇 시간 안에 생산적으로 쓸 수 있다.
- **SBOM**: Rust(cargo-cyclonedx)와 JavaScript(cyclonedx-npm)용 SBOM을 저장소에서 제공.

## 관련 노트

- [[Amazon CloudWatch]] — AWS의 관측성/모니터링 서비스
- [[멀티테넌시]] — 조직·스트림 단위 데이터 격리의 기반 개념
- [[고가용성]] — HA 모드가 지향하는 무중단 운영 속성
- [[bytedance-deer-flow]] — OpenTelemetry(Monocle) 기반 에이전트 관측을 지원하는 하네스

> 원문: [openobserve/openobserve: Open source observability platform for logs, metrics, traces, frontend monitoring, pipelines and LLM observability](https://github.com/openobserve/openobserve)
> 원본 클립: [[2026-07-14-openobserveopenobserve Open source observability platform for logs, metrics, traces, frontend monitoring, pipelines and LLM observability. A sophisticated, simple and highly performant alternative to Datadog, Splunk, and Elasticsearch]]
