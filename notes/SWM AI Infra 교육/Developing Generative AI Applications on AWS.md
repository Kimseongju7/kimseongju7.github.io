---
title: Developing Generative AI Applications on AWS
date: 2026-07-10
tags: [aws, 생성형ai, bedrock, swm, rag, 프롬프트엔지니어링]
categories: [학습, aws]
publish: false
---
SWM AI Infra 교육 "Developing Generative AI Applications on AWS" 강의 정리 노트. Amazon Bedrock을 중심으로 프롬프트 엔지니어링, RAG, 평가, 책임감 있는 AI, 에이전트까지 생성형 AI 애플리케이션 개발 전 과정을 2일에 걸쳐 다룬다.

---

## Module 0: 과정 소개

강사가 직접 제작한 자료(AWS 공식 문서 아님)로 진행되는 과정이다. Day 1과 Day 2로 나뉘어 있고, Day 1은 생성형 AI 기초와 Bedrock API·프롬프트 엔지니어링·RAG를, Day 2는 오픈소스 프레임워크 연동, 평가, 책임감 있는 AI, 에이전트 개발을 다룬다. 실습 중심으로 구성되어 있어 각 모듈 뒤에 핸즈온 랩이 붙는다.

---

## Module 1: AWS 생성형 AI 애플리케이션 구성 요소

### 전통적 프로그래밍 vs 머신러닝
전통적 프로그래밍은 개발자가 규칙을 하나하나 작성하지만, 머신러닝은 컴퓨터가 데이터에서 직접 패턴을 학습해 예측 모델을 만든다. 이 관계를 확장하면 인공지능 ⊃ 머신러닝 ⊃ 딥러닝 ⊃ 생성형 AI 순서의 포함 관계가 된다. 생성형 AI는 방대한 데이터로 사전훈련한 모델이 텍스트·이미지·영상 같은 새로운 콘텐츠를 만들어내는 영역이다.

생성형 AI가 주는 이점은 크게 세 갈래다. 고객 경험 개선, 업무 자동화를 통한 효율성 향상, 그리고 새로운 비즈니스 인사이트나 제품 혁신이다.

### 파운데이션 모델
파운데이션 모델은 레이블이 없는 방대하고 다양한 데이터로 사전훈련한 딥러닝 모델이다. 입출력 형태에 따라 크게 네 갈래로 나뉜다.

- **Language Model** — text-to-text. LLM과 SLM이 여기 속한다.
- **Diffusion Model** — text-to-image. 이미지를 생성한다.
- **Embedding Model** — text-to-vector. 텍스트를 벡터로 바꾼다.
- **Multimodal Model** — 텍스트·이미지·오디오·영상을 동시에 다룬다.

### LLM과 컨텍스트 윈도우
LLM은 Attention 메커니즘을 쓰는 Transformer 구조를 기반으로 한다. 파라미터 수가 수십억~수조 개면 LLM, 수백만~수억 개면 SLM(Small Language Model)으로 구분한다.

컨텍스트 윈도우는 모델이 한 번에 처리할 수 있는 토큰 분량이다. `max_new_tokens` 같은 추론 파라미터로 응답 길이를 제어한다. 컨텍스트 윈도우를 키우면 정확도가 오르고 할루시네이션이 줄고 일관성이 좋아지지만, 그만큼 연산 비용이 늘고 프롬프트 공격에 노출될 여지도 커진다.

Stable Diffusion 같은 diffusion 모델은 실제 픽셀 공간이 아니라 압축된 latent 공간에서 연산해 훈련과 생성 속도를 끌어올린다.

### 애플리케이션 아키텍처 예시
생성형 AI 애플리케이션은 보통 프론트엔드(S3 + CloudFront), 백엔드(API Gateway + Lambda), 모델 호출(Amazon Bedrock)로 이어지는 [[서버리스]] 구조로 짜인다. 서비스 계층(실제 활용 애플리케이션) — 개발 도구 계층 — 인프라 계층(모델 훈련·배포 파이프라인)의 3단 구조로 이해하면 된다.

---

## Module 2: Amazon Bedrock을 이용한 프로그래밍

### LLM에게 보내는 요청 구성
클라이언트가 LLM에 보내는 요청은 세 요소로 이뤄진다. 사용자가 직접 입력하는 프롬프트, AI의 어조·성격·행동 경계를 규정하는 시스템 프롬프트, 그리고 필요하면 RAG로 덧붙이는 외부 리소스 정보다.

### 추론 파라미터
응답의 성격을 조절하는 값들이다.

- **Temperature (0.0~1.0)** — 낮으면 일관되고 보수적인 답, 높으면 창의적인 답.
- **Top-p (0.0~1.0)** — 누적 확률 기준으로 후보를 자른다. 분석 작업은 0.1~0.3, 창작은 0.7~0.9 정도를 쓴다.
- **Top-k** — 확률 상위 k개 단어 중에서만 고른다.
- **stop_sequences** — 특정 문자열이 나오면 생성을 멈춘다.
- **max_tokens** — 응답의 최대 길이.
- **Repetition Penalty** — 같은 표현이 반복되는 걸 억제한다.

### Bedrock API 두 갈래
Bedrock API는 컨트롤 플레인과 데이터 플레인으로 나뉜다. 컨트롤 플레인은 `ListFoundationModels`(모델 목록 조회), `CreateModelInvocationJob`(배치 작업), `CreateGuardrail`(안전 정책 생성)처럼 리소스 관리용이고, 데이터 플레인은 실제 추론을 수행하는 `InvokeModel`, `InvokeModelWithResponseStream`, `Converse`, `ApplyGuardrail`이다.

### InvokeModel vs Converse
InvokeModel은 텍스트·이미지·임베딩 등 범용 추론에 쓰고 모델마다 요청·응답 형식이 다르다. Converse는 대화형 애플리케이션을 위해 턴 구조를 표준화한 API로, 문서 첨부나 도구 호출까지 일관된 형식으로 다룰 수 있다. 즉시 응답이 필요하면 InvokeModel, 긴 텍스트를 점진적으로 보여주려면 InvokeModelWithResponseStream, 멀티턴 대화라면 Converse를 쓰는 식으로 상황에 맞게 골라 쓰면 된다.

---

## Module 3: 개발자를 위한 프롬프트 엔지니어링

파운데이션 모델은 일반적인 도메인의 데이터로 훈련되어 있어서, 특정 목적에 맞추려면 프롬프트 엔지니어링·RAG·에이전트·파인튜닝·자체 모델 개발 중 하나를 골라야 한다. 이 중 가장 손쉬운 커스터마이징이 프롬프트 엔지니어링이다.

### 프롬프트의 구성 요소
좋은 프롬프트는 다음을 담는다: 태스크(목표), 지침(수행 방법), 입력(고려할 정보), 출력(원하는 형식), 어조와 스타일, 제약 조건(피해야 할 것들). 시스템 프롬프트는 이 중 어조·성격·경계를 세션 전체에 걸쳐 고정하는 역할을 한다.

### 프롬프트 기법
- **Zero-Shot** — 예시 없이 충분한 컨텍스트만으로 요청한다.
- **Few-Shot** — 예시를 몇 개 곁들여 정확도를 높인다. 예시의 품질이 결과를 좌우한다.
- **Chain of Thought(CoT)** — 복잡한 문제를 여러 논리 단계로 나눠 풀도록 유도한다. 수학이나 절차 설명에 잘 맞는다.
- **Self Consistency** — CoT를 여러 번 돌려 답을 여러 개 뽑은 뒤 다수결로 최종 답을 정한다.
- **Tree of Thought(ToT)** — 중간 단계를 여러 갈래로 탐색하며, 전문가 관점을 활용해 탐구적이거나 전략적인 문제에 대응한다.
- **ReAct(Reasoning & Acting)** — 추론과 행동을 섞는다. Thought → Action → Observation을 반복하며 복잡한 문제를 풀어간다.

이 밖에 재사용 가능한 프롬프트 템플릿, 부적절한 응답을 막는 네거티브 프롬프트, 특정 전문가 관점을 부여하는 역할 기반 프롬프트도 실무에서 자주 쓰는 기법이다.

---

## Module 4: Amazon Bedrock API 활용

생성형 AI 애플리케이션이 흔히 처리하는 작업은 생성(텍스트·코드·이미지·오디오·영상), 요약, 질의응답, 대화(챗봇) 네 가지로 정리할 수 있다.

### Bedrock의 추론 모드
1. **배치 추론** — 대규모 데이터셋을 한 번에 처리한다. S3·Glue·Step Functions·EventBridge와 함께 쓴다.
2. **[[온디맨드]] 추론** — 요청이 오면 즉시 응답하는 대화형 방식. Lambda·ALB·EC2·DynamoDB·OpenSearch와 조합한다.
3. **프로비저닝된 처리량** — 모델 단위(MU)를 1개월 또는 6개월 약정으로 구매해 일정 처리량을 보장받는다.
4. **프롬프트 캐싱** — 프롬프트에 캐시 체크포인트를 넣어두면, TTL 안에 같은 요청이 다시 오면 캐시된 컨텍스트를 재사용한다. Nova, Claude(Sonnet·Haiku·Opus) 등이 지원하며 최소 1024~4096토큰이 필요하다.
5. **추론 프로파일** — 여러 리전의 용량을 묶어 쓰는 방식. 특정 리전 용량이 부족하면 다른 리전으로 자동 라우팅한다.

### 메모리로 컨텍스트 확장하기
단기 메모리는 한 대화 세션 안의 프롬프트·응답을 저장하는 것이고, 장기 메모리는 DynamoDB 같은 저장소에 대화 기록을 쌓아뒀다가 이후 대화에서 다시 불러오는 방식이다. 예를 들어 Lambda 기반 챗봇이라면 JWT에서 사용자 ID를 뽑고 → DynamoDB에서 이전 대화 기록을 조회하고 → Converse API로 응답을 생성한 뒤 → 그 결과를 TTL 30일짜리 레코드로 다시 DynamoDB에 저장하는 흐름을 탄다.

---

## Module 5: RAG로 생성형 AI 응답 커스터마이징

RAG(Retrieval-Augmented Generation)는 외부 지식 소스에서 프롬프트와 관련된 정보를 검색해 오고, 그 정보를 프롬프트에 덧붙여 답을 생성하는 방식이다.

### 지식 기반 만들기 (하이드레이션)
1. **수집·전처리** — 웹페이지·PDF 등에서 문서를 크롤링하고, 중복·헤더·푸터 같은 노이즈를 제거하고, OCR이나 PDF 로더로 텍스트를 뽑아 Markdown으로 정리하고, 언어를 식별한다.
2. **청킹(문서 분할)** — 방식에 따라 특성이 갈린다.
   - **고정 크기 청킹** — 균일한 크기로 자른다. 구현이 간단해 소규모 데이터셋에 적합하다.
   - **구조 인식 청킹** — Markdown·HTML·LaTeX 구조를 고려해 자른다. 기술 문서나 매뉴얼에 맞는다.
   - **시맨틱 청킹** — 코사인 유사도로 의미 경계를 찾아 자른다. 보고서, 논문처럼 의미 단위가 중요한 문서에 쓴다.
   - **재귀적 청킹** — 계층 구조를 만들며 다양한 크기를 지원한다. 복잡한 구조의 문서에 적합하다.

### 검색: k-NN과 ANN
- **k-NN** — 모든 벡터와 거리를 계산해 정확한 최근접 이웃을 찾는 정확 검색. 소규모 데이터셋에서만 현실적이다.
- **ANN** — 정확도를 조금 희생하는 대신 속도를 얻는 근사 검색. 대규모 데이터셋에서는 필수다.
  - **HNSW(그래프 기반)** — 유사한 벡터끼리 다층 그래프로 연결한다. 빠르고 정확하지만 메모리를 많이 먹는다(10억 개·128차원 기준 약 1,408GB).
  - **IVF(공간 분할 기반)** — 벡터를 클러스터링해 버킷으로 나눈 뒤 일부 버킷만 검색한다. HNSW보다는 메모리를 아낀다(약 1,126GB).
  - **Product Quantization** — 벡터를 하위 벡터로 쪼개 압축한다. 메모리를 극적으로 줄일 수 있어(약 70GB) 모바일·IoT 환경에 적합하다.
  - **하이브리드 검색** — 키워드 검색과 시맨틱 검색을 함께 쓴다.

### 오케스트레이터
검색된 문서와 사용자 쿼리를 보고 다음에 뭘 할지 결정하는 "브레인" 역할이다. 검색 결과의 순위를 재조정해 관련성 높은 정보를 앞에 배치하며, Lambda 함수나 Bedrock 지식 기반으로 구현한다.

### Amazon Bedrock 지식 기반 구축 절차
1. 지원 형식의 데이터 소스, 저장소, IAM 역할, 파운데이션 모델 액세스 권한을 준비한다.
2. `CreateKnowledgeBase`로 지식 기반을 생성한다. 임베딩 모델마다 지원하는 벡터 타입·차원이 다르고, 이미지·표 같은 멀티모달 데이터를 다루려면 S3 보조 저장소가 필요하다.
3. `CreateDataSource`로 데이터 소스 커넥터를 구성한다. 청킹 전략은 `FIXED_SIZE`, `HIERARCHICAL`, `SEMANTIC`, `NONE` 중에서 고르고, 데이터 소스는 S3·웹·Salesforce·SharePoint·커스텀·Redshift 메타데이터를 지원한다.
4. 벡터 저장소(OpenSearch, Aurora/RDS pgvector, Neptune ML, MemoryDB, DocumentDB 등)를 설정한다.
5. `StartIngestionJob`으로 동기화한다. 지속적인 동기화가 필요하면 별도 로직을 추가해야 한다.

마지막으로 `RetrieveAndGenerate` API는 Retrieve(검색)와 InvokeModel(생성)을 하나로 묶어, 관련 데이터를 검색하고 인용을 포함한 자연어 응답까지 한 번에 만들어준다.

---

## Module 6: 오픈소스 프레임워크와 Amazon Bedrock 통합

Bedrock API는 완전관리형 서버리스 서비스라 AWS가 인프라를 전담하고 단일 API로 여러 벤더 모델을 쓸 수 있는 대신, 커스터마이징은 제공 범위 안에서만 가능하다. 반대로 오픈소스 프레임워크는 개발자가 직접 설치·운영해야 하는 부담이 있지만 소스 코드를 자유롭게 고칠 수 있다. 두 접근을 필요에 따라 섞어 쓰는 게 실무적이다.

### LangChain
LLM 애플리케이션 개발에 특화된 오픈소스 프레임워크로, LLM을 외부 데이터·환경과 연결하는 역할을 한다.

- **프롬프트 템플릿** — 변수를 채워 넣는 구조화된 프롬프트 문자열.
- **체인** — LLM과 다른 에이전트·도구를 순서대로 엮어 RAG나 에이전트를 구현한다.
- **인덱스** — 문서 로더, 벡터 DB, 텍스트 스플리터 등 외부 리소스를 관리한다.
- **메모리** — 대화 전체 혹은 요약본을 저장해 모델이 맥락을 기억하게 한다.
- **에이전트** — LLM을 추론 엔진 삼아 다른 데이터소스·도구를 조합해 작업을 스스로 결정한다.

실제로는 DynamoDB 세션 테이블에 대화 기록을 저장하고 `RunnableWithMessageHistory`로 메시지 히스토리를 자동 관리하는 식으로 메모리가 붙은 체인을 구성한다.

---

## Module 7: 생성형 AI 애플리케이션 구성 요소 평가

### 파운데이션 모델 평가
정량적으로는 Vellum LLM Leaderboard, HuggingFace MMLU-Pro 같은 벤치마크로 객관적 성능을 본다. 정성적으로는 실제 사용 사례를 반영한 테스트 프롬프트를 만들어 비용·품질·속도를 종합적으로 저울질하며, HITL(Human-In-The-Loop)이나 LLM 자체를 평가자로 쓰기도 한다.

### Amazon Bedrock의 모델 평가
- **자동화 평가** — 텍스트 생성, 요약, Q&A, 분류 작업에 대해 정확성·견고성·유해성 같은 지표를 빌트인 또는 자체 JSON 데이터셋으로 측정한다.
- **인적 평가** — 단일 모델 평가(좋아요/싫어요, 리커트 척도) 또는 두 모델 비교 평가(선택형, 서수 랭크)를 지원한다. SageMaker Ground Truth로 평가팀을 꾸려 이메일로 초대하는 식으로 운영한다.
- **LLM 평가** — 사람이 하던 평가를 LLM이 대신한다. Generator 모델과 Evaluator 모델을 따로 두고, 유용성·정확성·충실함·전문성/어조·완전성·일관성·관련성·지침 준수·가독성 같은 품질 지표와 유해성·거절·스테레오타이핑 같은 책임 AI 지표로 채점한다.

### RAG 평가 (Ragas)
RAG는 검색이 잘 됐는지, 검색된 컨텍스트로 답변을 잘 만들었는지 두 축을 모두 봐야 한다. Ragas(Retrieval-Augmented Generation Assessment)는 이를 위한 전용 평가 프레임워크로, `user_input`·`retrieved_contexts`·`response`·`reference`를 담은 평가 샘플을 만들어 지표를 계산한다.

- **검색 성능** — Context Precision(답변 관련 문서 수/전체 검색 문서 수), Context Recall(검색된 관련 문서/전체 관련 문서 수), Context Entity Recall
- **답변 품질** — Faithfulness(검색 컨텍스트에 얼마나 충실한지, 즉 할루시네이션 억제), Answer Relevancy(답변에서 역으로 질문을 만들어 원래 질문과 비교), Answer Similarity, Answer Correctness
- **안정성** — Harmfulness, Maliciousness, Coherence, Correctness, Conciseness

Bedrock도 지식 기반 검색 단계(정밀도·재현성)와 검색+생성 단계(유용성·정확성·논리적 일관성·충실도·완전성·인용 정밀도·인용 범위 등)를 나눠 RAG를 평가하는 기능을 제공한다.

### 지연 시간·비용 최적화
프롬프트 캐싱(캐시 체크포인트 TTL 5분, 문서 Q&A·긴 지침·대화형 에이전트·코딩 도우미에 유용), 지능형 프롬프트 라우팅(품질 대비 비용을 계산해 같은 벤더의 여러 모델 중 최적 모델로 자동 라우팅), 지연 시간 최적화 추론(TTFT·OTPS·E2E 같은 지표 개선)을 함께 쓰면 비용과 응답 속도를 조율할 수 있다.

---

## Module 8: 책임감 있는 AI 구현

책임감 있는 AI는 사용자 안전과 데이터 보안 위험을 최소화하도록 AI를 설계·개발·운영하는 실천이다. 할루시네이션 방지, 유해성 방지, 지적재산권·개인정보 보호, 신뢰와 규정 준수 확보가 목표다.

### 편향의 유형
- **훈련 데이터의 편향** — 성별·인종 편향(오분류 사건), 데이터 다양성 부족(제한된 조건에서만 학습), 사회적·역사적 편향(검색 엔진이 고정관념을 강화하는 사례)
- **프롬프트 자체의 편향** — 유도성 질문, 특정 관점 강요("최고의 해결책" 같은 표현), 불균형한 예시 제시, 모호하고 주관적인 평가 기준 요구
- **편향 증폭** — 알고리즘이 초기 편향을 반복하며 점점 키우는 현상

### 적대적 프롬프팅
- **Prompt Injection** — 입력에 악의적 명령을 끼워 넣는 것
- **Jail Breaking** — 안전장치를 우회하려는 시도
- **Prompt Leaking** — 시스템 프롬프트를 노출시키는 것
- **Social Engineering** — 사회공학적 방법으로 정보를 캐내는 것

### Amazon Bedrock Guardrails
단어 필터(비속어 등 차단), 콘텐츠 필터(증오·모욕·성적·폭력·불법행위·프롬프트 공격), 거부된 주제, 컨텍스트 근거 확인(소스와 쿼리를 대조해 할루시네이션 감지), 민감 정보 필터(PII·커스텀 정규식 차단·마스킹), 자동 추론 검사(정책·규칙 기반으로 응답 정확성 검증)까지 다층으로 필터링한다.

Standard Tier와 Classic Tier의 차이도 알아둘 만하다. Standard Tier는 콘텐츠 필터·프롬프트 공격 방어가 더 견고하고, 거부된 주제를 최대 1,000자까지 지원하며, 언어 지원 범위가 넓고, 교차 리전 추론과 프롬프트 누수 감지를 지원한다. Classic Tier는 영어·프랑스어·스페인어만 지원하고 거부된 주제는 200자까지다.

---

## Module 9: 생성형 AI 애플리케이션의 도구와 에이전트

**AI Agent**는 외부 지식 기반과 시스템을 호출·연동해 클라이언트 입력을 처리하는 시스템이고, **Agentic AI**는 목표를 스스로 파악·계획·실행하는 자율적인 시스템이다.

### 오픈소스 에이전트 프레임워크 비교
- **LangGraph** — 복잡한 상태 관리와 사이클을 지원해 정교한 오케스트레이션에 강하다.
- **CrewAI** — 역할 기반 협업이 쉬워 팀 시뮬레이션이나 보고서 생성에 적합하다.
- **LlamaIndex** — RAG 통합에 특화돼 문서 분석·검색에 강하다.
- **Strands Agents** — AWS 네이티브에 스트리밍을 지원해 프로덕션 환경에 적합하다.

**Strands Agents**는 AWS가 만든 Python 기반 오픈소스 프레임워크로, MCP 프로토콜로 도구를 다루고 AWS 인증·권한 관리와 잘 맞물린다. 모델(Bedrock·Claude·Llama 등) → 도구(MCP 서버 또는 직접 만든 Python 도구) → 프롬프트(자연어 지시 + 시스템 프롬프트)를 조합해, 입력 수신 → LLM 처리 → 도구 실행 → 결과 통합 → 응답 생성이라는 Agent Loop를 돈다.

**CrewAI**는 2024년 초 등장한 프로덕션급 에이전트 프레임워크로 GUI(CrewAI UI Studio)도 지원한다. Crew(여러 에이전트의 협업 오케스트레이션), Agent(역할을 맡은 AI), LLM(에이전트의 지능), 메모리(단기·장기·엔터티·외부), 도구, Task(구체적 과제), Flows(조건문·반복문·분기로 짠 워크플로우) 개념으로 구성된다.

**LangChain vs LangGraph**는 역할이 다르다. LangChain은 모델·구성요소의 표준 인터페이스로 단순한 체인이나 검색 흐름에 적합하고, LangGraph는 Graph(Node+Edge), State(현재 상태), Node(LLM 호출·도구 실행 등의 로직), Edge(조건 분기 포함 연결), HITL(사람의 개입·승인)을 갖춘 스테이트풀 오케스트레이션으로 복잡한 에이전틱 시스템에 적합하다.

### MCP와 A2A
- **MCP(Model Context Protocol)** — Client(AI 애플리케이션을 외부 도구·데이터 소스에 연결)와 Server(LLM이 외부 서비스·DB·API와 연동)로 구성된다.
- **A2A(Agent2Agent) 프로토콜** — 서로 다른 프레임워크로 만든 에이전트끼리 통신하는 표준이다. AgentCard(`/.well-known/agent.json`)로 협력 대상을 등록·탐색하고, `tasks/send`나 `sendSubscribe`로 작업을 요청하고, SSE나 Webhook으로 진행 상황을 받다가, `input-required` 상태면 추가 입력을 보내고, 최종 결과는 Artifact로 돌려받는 흐름을 탄다.

---

## Module 10: Amazon Bedrock 에이전트 개발

### Amazon Bedrock Flows
시각적 빌더로 생성형 AI 애플리케이션의 로직(멀티턴 대화, 프롬프트 템플릿 관리, 가드레일 적용)을 구성하고 오케스트레이션하는 기능이다. `invoke_flow()` API로 입력을 넣으면 각 노드의 처리 결과가 스트림으로 돌아온다.

### Agents vs Inline Agents
- **Agents** — 런타임 이전에 컨트롤 플레인에서 미리 정의해두는 방식. `InvokeAgent`로 호출하며, 일관되고 반복 가능한 작업에 적합하지만 재구성하려면 버전 관리가 필요하다.
- **Inline Agents** — API 호출 시점에 동적으로 정의하는 방식. `InvokeInlineAgent`로 호출하며, 별도 버전 관리 없이 동적인 임시 작업이나 빠른 실험에 적합하다.

에이전트는 Action Group(외부 도구를 호출하는 API 설정), Knowledge Base, Foundation Model, Code Interpreter(안전한 코드 실행 환경), Memory(멀티세션 대화 유지) 같은 구성 요소로 이뤄진다.

### Multi-Agent Collaboration
협업자 에이전트가 개별 로직을 처리하고, 감독자 에이전트가 질문을 확인해 적절한 워커 에이전트로 라우팅하는 구조다.

### Amazon Bedrock AgentCore
고성능 AI 에이전트를 안전하게 대규모로 배포·운영하기 위한 구성 요소 묶음이다.

- **SDK & Runtime** — 비동기 처리와 최대 8시간 장기 실행을 지원하고, MCP 서버를 대규모로 돌릴 수 있다.
- **Gateway** — Lambda 함수나 오픈 API 엔드포인트를 MCP로 노출시켜 준다. OAuth 2.0/OIDC 인증과 시맨틱 검색을 지원한다.
- **ID** — 에이전트용 ID·자격증명 관리 서비스로, 인바운드·아웃바운드 인증과 권한부여를 담당한다.
- **Memory** — 시맨틱·요약·사용자 커스텀 메모리를 지원하고, 세션 단위 대화 턴을 영구 저장한다.
- **Tools** — Browser(웹 검색·정보 추출), Code Interpreter(코드 실행·분석) 같은 내장 도구를 제공한다.
- **Observability** — 에이전트 동작을 관찰할 수 있는 기능을 제공한다.

---

## 실습

1. **실습 1: Amazon Bedrock API를 사용한 개발** — InvokeModel 등 기본 Bedrock API로 모델을 직접 호출해본다.
2. **실습 2: Amazon Bedrock API를 사용한 스트리밍 패턴 개발** — InvokeModelWithResponseStream으로 응답을 점진적으로 받아오는 스트리밍 패턴을 구현한다.
3. **실습 3: Amazon Bedrock API를 사용한 대화 패턴 개발** — Converse API와 메모리를 활용해 멀티턴 대화형 애플리케이션을 만든다.
4. **실습 4.1: RetrieveAndGenerate API와 함께 완전관리형 RAG** — Bedrock 지식 기반과 RetrieveAndGenerate API로 완전관리형 RAG 파이프라인을 구성한다.
5. **실습 4.2: Retrieve API와 함께 Amazon Bedrock Knowledge Bases를 사용한 Q&A** — Retrieve API로 검색만 따로 수행해 직접 답변 생성 로직을 붙여보는 Q&A 실습이다.
6. **실습 4.3: RAG 평가(RAGAS) 프레임워크를 사용한 Q&A 애플리케이션** — Ragas로 RAG 기반 Q&A 애플리케이션의 검색·답변 품질을 정량 평가한다.
7. **실습 4.4: RetrieveAndGenerate API를 사용한 가드레일 기능 테스트** — Bedrock Guardrails를 적용해 RAG 응답의 안전성 필터링을 테스트한다.
8. **실습 5: Amazon Bedrock Agents 개발(캡스톤)** — Action Group·Knowledge Base 등을 조합해 실제로 동작하는 Bedrock 에이전트를 처음부터 구축하는 마무리 프로젝트다.

---

## 관련 노트
- [[사전강의 - AWS Cloud Practitioner Essentials]] — 같은 SWM AI Infra 교육에서 먼저 다룬 AWS 클라우드 기본 개념 강의
- [[Machine Learning Engineering on AWS]] — 같은 교육에서 SageMaker 기반 ML 파이프라인을 다룬 강의로, 이 노트는 그 뒤를 이어 생성형 AI·Bedrock 중심으로 확장된다
- [[서버리스]] — Module 1의 애플리케이션 아키텍처(S3+CloudFront, API Gateway+Lambda, Bedrock)가 따르는 서버리스 구조
- [[Amazon Bedrock AgentCore 워크샵]] — Module 10에서 소개한 AgentCore를 Runtime·Memory·Gateway 등 9개 랩으로 깊게 다룬 워크샵
- [[AgentCoreWithWebApp 빈칸 채우기 실습]] — 그 AgentCore 개념을 채팅 웹앱 하나로 통합한 빈칸 채우기 실습
- [[AI Infra 교육 정리본]] — 이 강의(4~5일차)를 포함한 SWM AI Infra 교육 5일 전체 정리본
