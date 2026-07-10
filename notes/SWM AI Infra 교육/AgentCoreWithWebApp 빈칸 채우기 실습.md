---
title: AgentCoreWithWebApp 빈칸 채우기 실습
date: 2026-07-10
tags:
  - aws
  - bedrock
  - agentcore
  - ai-agent
  - mcp
  - swm
categories:
  - 학습
  - aws
slug: agentcorewithwebapp-workshop
publish: false
---
[k2hdevil/AgentCoreWithWebApp-workshop](https://github.com/k2hdevil/AgentCoreWithWebApp-workshop) 실습 기록. [[Amazon Bedrock AgentCore 워크샵]]에서 다룬 Runtime·Memory·Gateway·Identity·Code Interpreter 개념을 하나의 실제 채팅 웹앱(FastAPI + 브라우저 UI)으로 통합한 프로젝트를, 코드 17곳을 빈칸으로 비운 "빈칸 채우기" 형식으로 변환한 워크샵이다. 로컬 클론: `/Users/rlatjdwn/projects/AgentCoreWithWebApp-workshop`.

---

## 프로젝트 개요

원본은 AWS 공식 샘플 [sample-amazon-bedrock-agentcore-onboarding](https://github.com/aws-samples/sample-amazon-bedrock-agentcore-onboarding)(=[[Amazon Bedrock AgentCore 워크샵]]의 Lab별 개별 예제 코드)이다. 이 저장소는 Lab 1(Code Interpreter)·Lab 2(Runtime)·Lab 3(Memory)·Lab 6(Identity)·Lab 7(Gateway) 다섯 개 Lab의 개별 코드를 하나의 실행 가능한 웹 채팅 애플리케이션으로 통합하고, AI 코딩 에이전트 **Kiro**로 빈칸 채우기(fill-in-the-blank) 워크샵 형식으로 변환한 뒤 HITL(Human-in-the-Loop)로 빈칸 난이도·힌트 정확성을 검수해 만들어졌다.

**아키텍처** (Runtime이 Memory·Gateway·Identity를 내부에서 오케스트레이션하고, 웹 백엔드는 얇은 프록시 역할만 한다):

```
Browser (web/static/index.html) — 채팅 UI, 세션 관리
        │ POST /api/chat
        ▼
Web Backend (web/app.py) — FastAPI, boto3 invoke_agent_runtime(), SSE 스트리밍
        │ invoke_agent_runtime API
        ▼
AgentCore Runtime (agent/invoke.py)
  ├── Memory: list_events() 이전 대화 조회 → create_event() 현재 대화 저장
  ├── Identity: Cognito client_credentials → Gateway Bearer 토큰
  ├── Agent(Strands + Claude): Code Interpreter, AWS Pricing MCP, Gateway MCP 도구 사용
  └── Gateway → Lambda(markdown_to_email) → SES 이메일 발송
```

실습은 5개 소스 파일에 총 **17개 TODO**(`"_____"` 문자형 빈칸 / `____` 숫자형 빈칸)를 채우는 방식으로 진행되며, 각 빈칸에는 `# HINT:` 주석과 공식 문서 링크가 달려 있다.

## 프로젝트 구조

```
.
├── agent/                              # AgentCore Runtime에 배포되는 코드
│   ├── invoke.py                       # [TODO 1~5] 엔트리포인트
│   ├── inbound_authorizer.json         # Cognito/Identity 설정 (setup_identity.py가 생성)
│   └── cost_estimator_agent/
│       ├── config.py                   # 시스템 프롬프트, 모델 설정
│       └── cost_estimator_agent.py     # [TODO 6~8, 15] 에이전트 핵심 로직
├── web/
│   ├── app.py                          # [TODO 9~11, 14] FastAPI 백엔드
│   └── static/index.html               # 채팅 프론트엔드
├── identity/
│   └── setup_identity.py               # [TODO 12~13] Cognito + Identity Provider 생성
├── gateway/
│   ├── setup_outbound_gateway.py       # [TODO 16~17] Gateway + Lambda Target 등록
│   ├── src/app.py                      # Lambda 함수 (markdown_to_email)
│   └── template.yaml                   # SAM 템플릿
└── solutions/                          # 정답 버전 전체 (워크샵 배포용, .gitignore 제외 대상)
```

## 진행 순서와 TODO 정답

### Step 1 — Identity (`identity/setup_identity.py`)
Cognito User Pool + App Client(M2M)를 만들고, AgentCore Identity에 OAuth2 자격증명 공급자로 등록한다.

- **TODO 13** (제어 플레인 boto3 서비스명): `boto3.client("bedrock-agentcore-control", region_name=region)`
- **TODO 12** (OAuth2 공급자 생성 API + vendor): `identity_client.create_oauth2_credential_provider(name=PROVIDER_NAME, credentialProviderVendor="CustomOauth2", oauth2ProviderConfigInput={...})`

```bash
cd identity && uv run python setup_identity.py
```

### Step 2 — 에이전트 핵심 로직 (`agent/cost_estimator_agent/cost_estimator_agent.py`)
Code Interpreter 세션을 시작하고 AWS Pricing MCP 도구를 연결해 Strands `Agent`를 구성한다.

- **TODO 15** (Code Interpreter 세션 시작): `self.code_interpreter.start()`
- **TODO 7** (Pricing MCP 서버 패키지명): `args=["awslabs.aws-pricing-mcp-server@latest"]`
- **TODO 6** (코드 실행 API): `self.code_interpreter.invoke("executeCode", {"language": "python", "code": calculation_code})`
- **TODO 8** (Agent 도구 목록 파라미터): `tools=all_tools`

### Step 3 — Gateway (`gateway/setup_outbound_gateway.py`)
Markdown→이메일 변환 Lambda를 SAM으로 배포하고, Gateway의 MCP Target으로 등록한다.

- **TODO 16** (Target 설정의 Lambda 키): `"targetConfiguration": {"mcp": {"lambda": {"lambdaArn": ..., "toolSchema": {"inlinePayload": tool_schema}}}}`
- **TODO 17** (아웃바운드 인증 방식): `"credentialProviderConfigurations": [{"credentialProviderType": "GATEWAY_IAM_ROLE"}]`

```bash
cd gateway
./deploy.sh your-verified-email@example.com   # Lambda 배포 (SES 발신자 인증 포함)
uv run python setup_outbound_gateway.py        # Gateway + Target 생성
```

### Step 4 — Runtime 엔트리포인트 (`agent/invoke.py`)
Memory·Gateway·Identity를 통합해 실제 요청을 처리하는 진입점을 완성한다.

- **TODO 1** (엔트리포인트 데코레이터): `@app.entrypoint`
- **TODO 2** (Memory 이전 대화 조회): `client.list_events(memory_id=..., actor_id=..., session_id=..., max_results=6)`
- **TODO 3** (Memory 현재 대화 저장): `client.create_event(memory_id=..., actor_id=..., session_id=..., messages=[(user_input, "USER"), (result, "ASSISTANT")])`
- **TODO 4** (M2M OAuth2 grant_type): `"grant_type": "client_credentials"`
- **TODO 5** (Gateway MCP 전송 계층): `streamablehttp_client(GATEWAY_URL, headers={"Authorization": f"Bearer {access_token}"})`

```bash
cd agent && uv run agentcore deploy --env AWS_REGION=us-west-2
```

### Step 5 — 웹 백엔드 (`web/app.py`)
배포된 Runtime을 호출하는 FastAPI 서버를 완성한다.

- **TODO 14** (데이터 플레인 boto3 서비스명): `boto3.client("bedrock-agentcore", config=config)`
- **TODO 9** (Runtime 호출 메서드): `client.invoke_agent_runtime(...)`
- **TODO 10** (에이전트 ARN 파라미터명): `agentRuntimeArn=config["agent_arn"]`
- **TODO 11** (세션 ID 파라미터명, 최소 33자): `runtimeSessionId=session_id` — `web/app.py`는 `webui_{timestamp}_{uuid4().hex}` 형태(54자)로 세션 ID를 생성해 이 제약을 만족시킨다.

```bash
cd web && uv run uvicorn app:app --reload --port 8080
```

### Step 6 — 동작 확인
`http://127.0.0.1:8080` 접속 후 순서대로 확인한다.
1. **Runtime**: "서울 리전의 EC2 t3.micro 24/7 비용 견적" → 기본 견적 응답 확인
2. **Memory**: 이어서 "버지니아 리전은 어떤가요?" → 이전 대화 맥락(서울 리전 비교 대상)을 기억하는지 확인
3. **Gateway**: "버지니아 리전과 서울 리전을 비교한 견적을 your-verified-email@example.com으로 보내주세요" → 이메일 수신 확인

## 빈칸 채우기 실습에서 짚어볼 포인트

- **Memory는 자동 저장이 아니다**: Runtime harness가 저장을 자동 처리해줄 거라 생각하기 쉽지만, 실제로는 `list_events`로 조회한 이력을 프롬프트에 직접 주입하고 `create_event`로 저장하는 것까지 개발자가 수동으로 구현해야 한다(`agent/invoke.py`의 `_retrieve_history`/`_store_conversation`).
- **세션 ID 33자 제약**: `runtimeSessionId`는 AgentCore Runtime이 최소 33자를 요구하므로, `web/app.py`는 타임스탬프+UUID4 hex를 조합해 54자짜리 ID를 만든다. 같은 세션 ID를 재사용해야 Memory가 대화를 하나의 맥락으로 묶는다.
- **Gateway 연동은 best-effort**: `agent/invoke.py`는 Gateway 토큰 취득이나 MCP 연결이 실패해도 예외를 던지지 않고 Gateway 없이(Code Interpreter + Pricing MCP만으로) 계속 동작하도록 방어적으로 짜여 있다 — 외부 연동 실패가 핵심 기능(비용 견적)까지 막지 않게 하는 설계.
- **`streamablehttp_client` 이름 이슈**: [[Amazon Bedrock AgentCore 워크샵]] 노트에도 적었듯, 이 이름은 MCP Python SDK v1 표기다. 최신 v2 SDK를 쓰는 환경이면 임포트 경로(`mcp.client.streamable_http`)와 이름이 달라질 수 있으니 `requirements.txt`의 `mcp` 버전을 확인해야 한다.

## 참고
- 원본 리포지토리: https://github.com/k2hdevil/AgentCoreWithWebApp-workshop
- 원본 샘플 코드: https://github.com/aws-samples/sample-amazon-bedrock-agentcore-onboarding
- 개념 배경: [[Amazon Bedrock AgentCore 워크샵]] (Lab 2 Runtime, Lab 3 Memory, Lab 6 Identity, Lab 7 Gateway에 각각 대응)
