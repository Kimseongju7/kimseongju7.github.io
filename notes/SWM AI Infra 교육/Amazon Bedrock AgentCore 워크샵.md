---
title: Amazon Bedrock AgentCore 워크샵
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
  - AWS
slug: agentcore-workshop
publish: false
---
SOMA - Agentic AI AgentCore Workshop("Amazon Bedrock AgentCore 워크샵: 기본부터 고급 에이전트 개발까지") 정리본. "AWS 비용 견적" 에이전트를 예제로 삼아 Code Interpreter·Runtime·Memory·Observability·Evaluation(Foundation 트랙)과 Identity·Gateway·Policy·Browser Use(Extension 트랙)까지 총 9개 랩을 점진적으로 확장하며 AgentCore의 각 기능을 익힌다.

---

## 워크샵 개요

이 워크샵은 AI 에이전트의 기본 개념부터 프로덕션 준비 구현까지 체계적이고 점진적인 접근 방식으로 다룬다. 처음부터 관찰성(Observability)과 평가(Evaluation)를 통합하는 최신 모범 사례를 반영해 설계되었으며, AI 에이전트를 구축하는 방법뿐 아니라 에이전트 프로젝트를 체계적으로 접근하는 방법을 배우고자 하는 엔지니어·아키텍트를 대상으로 한다.

전체 실습은 "AWS 비용 견적"을 공통 사용 사례로 삼는다. 정보에 접근해 계산 결과를 집계하는 이 패턴은 매출 데이터 분석이나 거래 요약 같은 다른 시나리오에도 폭넓게 적용할 수 있다. 구현은 Python 스크립트 기반이라 빠르게 시작할 수 있고, 부록 Lab A-1에서는 AI 코딩 에이전트 Kiro로 자신만의 에이전트를 직접 만들어보는 실습도 포함된다.

**두 가지 트랙 구성**
- **Foundation (Lab 1~5)** — 신뢰할 수 있는 에이전트 구축: Code Interpreter, Runtime, Memory, Observability, Evaluation
- **Extension (Lab 6~9)** — 외부 시스템 연결: Identity, Gateway, Policy, Browser Use

**Lab 의존 관계**
```
Lab 1 (Code Interpreter)
├── Lab 2 (Runtime)
│   ├── Lab 4 (Observability)
│   ├── Lab 5 --online (Evaluation)
│   └── Lab 6 (Identity)
│       └── Lab 7 (Gateway)
│           └── Lab 8 (Policy)
├── Lab 3 (Memory)
├── Lab 5 --local/--ondemand (Evaluation)
└── Lab 9 (Browser Use)

Lab A-1 (Custom Agent) ← 독립적으로 수행 가능
```

### 왜 Amazon Bedrock AgentCore인가
AI 에이전트의 구현 프레임워크는 빠르게 진화한다. 2025년 기준으로도 LangChain, LangGraph, CrewAI, Mastra 등 다양한 프레임워크가 등장했고 GitHub 스타 수 추이를 보면 새 프레임워크도 빠르게 채택되고 있다. 프레임워크에 강하게 종속된 형태로 구현하면 기술 스택이 금세 구식이 되고 기술 부채가 쌓일 위험이 있다. 머신러닝 분야에서 잘 알려진 "Hidden Technical Debt in Machine Learning Systems" 논문도 데이터 파이프라인·운영·평가·모니터링 같은 주변 인프라가 충분한 리소스 없이 개발되면 유지 관리하기 어려운 부채로 남는 경향을 지적한다.

AgentCore는 **구현 독립적 런타임**과 **프로덕션 준비 기능의 오프로드**라는 두 축으로 이 문제를 방지한다. 구현 프레임워크나 언어에 관계없이 동작하도록 설계돼 있어 모범 사례가 바뀌어도 유연하게 대응할 수 있고, 인증·관찰성·메모리 같은 주변 기능을 직접 만들지 않고 클라우드로 오프로드함으로써 구현·운영 비용을 함께 줄인다.

### 사전 준비
- **필수 지식**: 기본 Python 프로그래밍, AWS 서비스에 대한 기본 이해
- **환경**: AWS 계정(Amazon Bedrock Claude 모델을 us-west-2에서 활성화), Python 3.11+ 및 `uv` 패키지 매니저, Git, AWS CLI, AWS SAM(Lab 4에만 필요)
- **예상 소요 시간**: 총 약 2시간, 랩당 10~20분
- **샘플 코드**: [sample-amazon-bedrock-agentcore-onboarding](https://github.com/aws-samples/sample-amazon-bedrock-agentcore-onboarding)
- 가격 정보는 [Amazon Bedrock AgentCore Pricing](https://aws.amazon.com/bedrock/agentcore/pricing/) 참고, 실습 후 리소스 정리는 마지막 요약 섹션 참고

AWS가 호스팅하는 워크샵 환경을 사용할 경우, VS Code Server가 설치된 EC2 인스턴스를 즉시 이용할 수 있다. 이벤트 대시보드의 Event Outputs에서 Code Editor URL로 접속한 뒤 Git 사용자 정보(`git config --global user.name/email`)와 기본 브랜치(`main`)를 설정하고, `/workshop` 디렉토리에 이미 준비된 샘플 코드에서 `uv sync`로 종속성을 설치하면 준비가 끝난다. AWS CLI 리전은 `aws configure set region us-west-2`로 고정해둔다. VS Code Server에는 AWS CLI, SAM CLI, Git, Python 3.11+, uv, Docker 및 관련 VS Code 확장(AWS Toolkit, Python, Docker, Live Server)이 이미 설치돼 있다.

---

## Lab 1: Code Interpreter — 계산하는 에이전트 🧮

**목표**: AI 에이전트의 기본 메커니즘을 이해하고, AI 에이전트가 취약한 정확한 연산 처리를 안전하게 수행하기 위한 AgentCore Code Interpreter 사용법을 익힌다.

AI 에이전트는 ①목표 달성을 위한 계획 수립 → ②필요한 도구 사용 → ③관찰 결과를 바탕으로 계획 진행 상황 업데이트, 이 세 단계를 반복한다. 비용 견적 시나리오에서는 "웹사이트 견적을 하고 싶다"는 목표에 대해 AWS Pricing MCP Server로 필요한 서비스 가격을 수집하고, 모든 가격이 모이면 Code Interpreter로 계산하는 흐름이 된다. 즉 에이전트를 만드는 데 필요한 것은 계획을 세우는 엔진 역할의 기반 모델(Claude 등)과, 정보를 얻거나 외부 시스템을 조작하기 위한 도구뿐이다.

구현에는 **Strands Agents**라는 프레임워크를 사용한다. 기반 모델의 성능을 살려 구현을 단순화하는 "모델 주도" 접근 방식이라 비교적 쉽게 에이전트를 만들 수 있다. 에이전트 코드는 기반 모델을 지정하는 `BedrockModel`, 가격 조회·계산 도구를 담은 `tools`, 에이전트 지시사항인 `SYSTEM_PROMPT` 세 가지로 구성된다. 도구를 모델에 전달할 때의 공통 형식이 **MCP**(Model Context Protocol)로, "AI 애플리케이션을 위한 USB-C"라고도 불린다 — 외부 시스템이 MCP를 지원하면 어떤 에이전트와도 바로 연동할 수 있다.

```python
all_tools = [self.execute_cost_calculation] + pricing_tools
agent = Agent(
    BedrockModel(
        boto_client_config=Config(
            read_timeout=900,
            connect_timeout=900,
            retries=dict(max_attempts=3, mode="adaptive"),
        ),
        model_id=DEFAULT_MODEL
    ),
    tools=all_tools,
    system_prompt=SYSTEM_PROMPT
)
```

> **`Agent(model, tools, system_prompt, ...)`** — Strands Agents SDK
> - 설명: 기반 모델·도구·시스템 프롬프트를 조합해 에이전트 인스턴스를 만든다.
> - 파라미터:
>   - `model` (`Model | str | None`): 추론에 쓸 모델 provider(`BedrockModel` 등) 또는 모델 ID 문자열. 기본값은 `BedrockModel()`.
>   - `tools` (`list | None`): 에이전트가 쓸 도구 목록(함수, 파일 경로, `ToolProvider` 등). 지정하지 않으면 전체 도구 사용 가능.
>   - `system_prompt` (`str | list[SystemContentBlock] | None`): 모델 동작을 지시하는 시스템 프롬프트.
>   - 이 외에도 `messages`, `callback_handler`, `conversation_manager`, `session_manager`, `hooks` 등 다수의 키워드 전용 파라미터가 있다.
> - 반환값: `Agent` 인스턴스.
>
> **`BedrockModel(*, boto_session=None, boto_client_config=None, region_name=None, endpoint_url=None, **model_config)`** — Strands Agents SDK
> - 설명: Amazon Bedrock을 추론 provider로 쓰는 모델 객체를 생성한다.
> - 파라미터:
>   - `boto_client_config` (`botocore.config.Config`, 선택): Bedrock Runtime boto3 클라이언트 설정(타임아웃·재시도 정책 등).
>   - `region_name` (`str`, 선택): 사용할 AWS 리전. 기본값은 `AWS_REGION` 환경 변수 또는 `us-west-2`.
>   - `**model_config`: `model_id` 등 Bedrock 모델 설정을 키워드 인자로 전달(예: `model_id="global.anthropic.claude-sonnet-4-6"`).
> - 반환값: `BedrockModel` 인스턴스(→ `Agent`의 `model` 인자로 전달).

**AgentCore Code Interpreter**는 AI가 생성한 코드를 완전히 격리된 샌드박스(microVM)에서 안전하게 실행하는 완전 관리형 서비스다. 메모리·CPU 상한으로 과도한 리소스 사용을 막고, 기본 15분에서 최대 8시간까지 장시간 실행을 지원하며, Python/JavaScript/TypeScript와 pandas·NumPy·Matplotlib 같은 주요 라이브러리가 사전 설치돼 있어 인프라 관리 없이 즉시 사용할 수 있다.

```python
from bedrock_agentcore.tools.code_interpreter_client import CodeInterpreter

# 1. 인스턴스 생성
interpreter = CodeInterpreter(region="us-west-2")

# 2. 세션 시작
interpreter.start()

# 3. 코드 실행
result = interpreter.invoke("executeCode", {
    "language": "python",
    "code": "print('Hello from secure sandbox!')"
})

# 4. 세션 종료
interpreter.stop()
```

> **`CodeInterpreter(region, session=None, integration_source=None)`** — `bedrock_agentcore.tools.code_interpreter_client`
> - 설명: AWS Code Interpreter 샌드박스와 통신하는 클라이언트를 생성한다.
> - 파라미터: `region` (`str`, 필수) — 사용할 AWS 리전. `session` (`boto3.Session`, 선택) — 커스텀 boto3 세션.
> - 반환값: `CodeInterpreter` 인스턴스.
>
> **`.start(identifier=DEFAULT_IDENTIFIER, name=None, session_timeout_seconds=900)`**
> - 설명: 샌드박스 세션을 시작한다.
> - 파라미터: `identifier`(`str`, 선택) — 사용할 인터프리터 식별자(기본은 시스템 기본 인터프리터). `session_timeout_seconds`(`int`, 기본값 `900`) — 세션 타임아웃(초), 기본 15분.
> - 반환값: 새로 생성된 세션의 `session_id`(`str`).
>
> **`.invoke(method, params=None)`**
> - 설명: 활성 세션이 없으면 자동으로 새 세션을 시작한 뒤, 샌드박스에서 지정한 메서드를 호출한다(예: `executeCode`로 코드 실행).
> - 파라미터: `method`(`str`, 필수) — 호출할 메서드 이름. `params`(`dict`, 선택) — 메서드 전달 파라미터(예: `{"language": "python", "code": "..."}`).
> - 반환값: 샌드박스 서비스의 응답(`dict`).
>
> **`.stop()`**
> - 설명: 현재 활성 세션을 종료한다(활성 세션이 없으면 아무 것도 하지 않고 성공 반환).
> - 반환값: `bool` — 성공 시 `True`.

실습에서 구현하는 `AWSCostEstimatorAgent`는 사용자 입력("빵집 웹사이트를 HTML로 최대한 저렴하게 만들고 싶다")에서 필요한 AWS 서비스를 추론(S3, CloudFront 등) → AWS Pricing MCP Server의 `get_pricing`으로 실시간 가격 조회(에러 발생 시 자동 재시도) → 가격 집계용 Python 코드를 LLM이 생성 → 생성된 코드를 Code Interpreter(`start()` → `invoke("executeCode", ...)` → `stop()`)에서 안전하게 실행해 결과를 반환하는 흐름으로 동작한다. `@contextmanager`로 성공·실패 여부와 무관하게 Code Interpreter가 확실히 종료되도록 구현돼 있다.

```python
@contextmanager
def _estimation_agent(self) -> Generator[Agent, None, None]:
    """Context manager for cost estimation components"""
    try:
        self._setup_code_interpreter()  # Code Interpreter 시작
        aws_pricing_client = self._setup_aws_pricing_client()

        # 에이전트 생성 및 처리 실행
        with aws_pricing_client:
            # ... 에이전트 처리 ...
            yield agent

    except Exception as e:
        logger.exception(f"❌ Component setup failed: {e}")
        raise
    finally:
        self.cleanup()  # Code Interpreter 중지를 포함한 정리
```

> **`@contextlib.contextmanager`** — Python 표준 라이브러리
> - 설명: 제너레이터 함수를 `with` 문에서 쓸 수 있는 컨텍스트 매니저로 바꾸는 데코레이터. `yield` 이전 코드가 진입(`__enter__`) 시, `yield` 이후(특히 `finally` 블록)가 종료(`__exit__`) 시 실행되어 예외 발생 여부와 무관하게 정리 코드가 실행되도록 보장한다.
> - 파라미터: 데코레이트 대상은 정확히 한 번 `yield`하는 제너레이터 함수여야 한다.
> - 반환값: `with` 문에서 사용 가능한 컨텍스트 매니저 팩토리 함수.

## Lab 2: AgentCore Runtime — 클라우드 배포 🚀

**목표**: Lab 1에서 로컬로 실행하던 에이전트를 Amazon Bedrock AgentCore Runtime을 이용해 클라우드에 배포한다.

AgentCore Runtime은 **AI 에이전트를 위한 [[서버리스]] 실행 환경**이다. 요청을 처리할 때만 작동하고(최대 8시간까지 장시간 작업 가능), 실제 런타임 시간만 과금되며(LLM 응답 대기 시간은 비과금), MCP 프로토콜로 접근 가능하다. 각 세션은 독립된 메모리·파일시스템·CPU를 가진 전용 microVM에서 실행돼 세션별로 기밀성이 보장되고, AgentCore Identity 통합으로 다양한 인증 방식을 지원하며, 처리 추적 기능이 내장돼 있다. 구현 언어나 프레임워크에 관계없이 특정 API를 구현한 Docker 컨테이너라면 무엇이든 실행할 수 있다는 점에서 "구현 독립적"이다.

배포에는 **Bedrock AgentCore Starter Toolkit**을 사용한다. `prepare_agent.py`로 IAM 역할을 생성하고 Lab 1의 소스 코드를 배포 디렉토리로 복사한 뒤, `agentcore configure`(진입점·실행 역할·종속성·리전을 지정해 `.bedrock_agentcore.yaml` 구성 파일 생성)와 `agentcore deploy`(Docker 이미지 빌드 → ECR 푸시 → AgentCore Runtime 환경 생성)로 배포한다. 진입점(`deployment/invoke.py`)은 `@app.entrypoint` 데코레이터로 감싼 매우 단순한 구현이며, 배포가 끝나면 AWS 콘솔의 "Amazon Bedrock AgentCore → Agent Runtime"에서 확인하거나 `agentcore invoke` 명령 또는 콘솔의 Test Endpoint로 직접 호출해볼 수 있다.

진입점 코드는 `BedrockAgentCoreApp`을 생성하고 `@app.entrypoint`로 감싼 함수 하나만 정의하면 된다 — 이 함수가 Runtime이 요청을 받을 때마다 실행된다.

```python
import sys
import os
sys.path.append(os.path.dirname(os.path.abspath(__file__)))
from cost_estimator_agent.cost_estimator_agent import AWSCostEstimatorAgent
from bedrock_agentcore.runtime import BedrockAgentCoreApp

app = BedrockAgentCoreApp()

@app.entrypoint
def invoke(payload):
    user_input = payload.get("prompt")
    agent = AWSCostEstimatorAgent()
    return agent.estimate_costs(user_input)

if __name__ == "__main__":
    app.run()
```

> **`BedrockAgentCoreApp(debug=False, lifespan=None, middleware=None)`** — `bedrock_agentcore.runtime`
> - 설명: AgentCore Runtime에 배포 가능한 애플리케이션(Starlette 기반)을 생성한다. `/invocations`, `/ping`, `/ws` 라우트를 자동으로 등록한다.
> - 파라미터: `debug`(`bool`, 기본값 `False`) — 디버그용 작업 관리 기능 활성화 여부.
> - 반환값: `BedrockAgentCoreApp` 인스턴스.
>
> **`@app.entrypoint`**
> - 설명: 함수를 Runtime의 메인 진입점으로 등록하는 데코레이터. Runtime이 요청(`/invocations`)을 받을 때마다 이 함수가 호출된다.
> - 파라미터: 데코레이트 대상 함수는 요청 본문을 담은 `payload`(`dict`)를 인자로 받는다.
> - 반환값: 원본 함수를 그대로 반환(추가로 `.run()` 메서드가 부착됨).

배포 명령은 `configure`(구성 파일 생성) → `deploy`(빌드·푸시·배포) → `invoke`(테스트 호출) 순으로 진행한다.

```bash
uv run agentcore configure --entrypoint deployment/invoke.py \
  --name cost_estimator_agent \
  --execution-role <execution-role-arn> \
  --requirements-file deployment/requirements.txt \
  --disable-memory \
  --non-interactive \
  --deployment-type direct_code_deploy \
  --region <region>

uv run agentcore deploy --env AWS_REGION=$(aws configure get region)

uv run agentcore invoke '{"prompt": "SSH 연결을 위한 작은 EC2 인스턴스 환경을 준비하고 싶습니다. 비용이 얼마나 들까요?"}'
```

> **`agentcore configure` 주요 옵션** — Bedrock AgentCore Starter Toolkit CLI
> - `--entrypoint, -e` (`str`): 진입점 파일 또는 디렉터리 경로.
> - `--name, -n` (`str`): 에이전트 이름.
> - `--execution-role, -er` (`str`): 사용할 IAM 실행 역할 ARN(없으면 새로 생성).
> - `--requirements-file, -rf` (`str`): 의존성 파일 경로.
> - `--disable-memory, -dm` (플래그): 메모리 기능 비활성화.
> - `--non-interactive, -ni` (플래그): 대화형 프롬프트 없이 기본값(또는 지정한 옵션값)으로 진행.
> - `--deployment-type, -dt` (`str`): `container` 또는 `direct_code_deploy`.
> - `--region, -r` (`str`): 배포할 AWS 리전.
> - 이 외 `--vpc`, `--subnets`, `--security-groups`, `--idle-timeout`, `--max-lifetime`, `--protocol` 등도 지원한다.
> - 동작: `.bedrock_agentcore.yaml` 구성 파일을 생성/갱신한다.
>
> **`agentcore deploy`**
> - 설명: `configure`로 만든 구성을 바탕으로 Docker 이미지를 빌드해 ECR에 푸시한 뒤, AgentCore Runtime 환경을 생성/갱신한다.
> - 참고: `--env KEY=VALUE`로 런타임 환경 변수를 지정할 수 있다.
>
> **`agentcore invoke '<JSON payload>'`**
> - 설명: 배포된 Runtime에 테스트 페이로드를 보내 응답을 확인한다.

## Lab 3: Memory — 비용 추정 에이전트에 기억 부여하기 🧠

**목표**: AgentCore Memory로 개인화된 경험을 위한 "기억" 기능을 구현한다.

AgentCore Memory는 **단기 기억**(세션 내 에이전트-사용자 상호작용을 이벤트로 저장, 대화 컨텍스트 유지)과 **장기 기억**(대화 요약, 확인된 사실, 사용자 선호도 등 세션을 넘어 이어져야 할 통찰 저장) 두 가지를 제공한다. **Memory Strategy**가 단기 기억을 장기 기억으로 전환하는 방식을 정의하며, 네 가지 유형이 있다: `SemanticMemoryStrategy`(사실·지식 중심), `UserPreferenceMemoryStrategy`(사용자 선호·선택 패턴 중심), `SummaryMemoryStrategy`(핵심 포인트·결정 중심), Episodic memory strategy(효과적이었던 행동 패턴 중심).

메모리 생성 시 `namespaces`로 기억을 논리적으로 그룹화한다(예: 고객 ID를 네임스페이스에 포함시켜 사용자 간 데이터 혼입 방지). 세분화 수준은 세션 수준 → 액터(사용자) 수준 → 전략 수준 → 글로벌까지 네 단계로 설계할 수 있다. 핵심 API는 세 가지다: `create_event`(대화를 단기 기억에 저장), `list_events`(단기 기억에서 이벤트 검색, 과거 견적 비교에 사용), `retrieve_memories`(장기 기억에 대한 의미론적 검색, 쿼리와 관련된 기억을 관련성 순으로 반환 — 예: 사용자가 소규모 아키텍처를 선호한다는 과거 대화 패턴을 추론해 그에 맞는 제안 생성). 이를 통해 에이전트는 사용자가 이전에 공유한 정보를 반복해서 묻지 않고도 개인화된 추천을 제공할 수 있다.

`UserPreferenceMemoryStrategy`로 장기 기억(사용자 선호도 추출)을 활성화하며 네임스페이스에 `actor_id`를 포함시킨다.

```python
def __init__(self, actor_id: str, region: str = "", force_recreate: bool = False):
    self.memory_client = MemoryClient(region_name=self.region)

    self.memory = self.memory_client.create_memory_and_wait(
        name=memory_name,
        strategies=[{
            "userPreferenceMemoryStrategy": {
                "name": "UserPreferenceExtractor",
                "description": "Extracts user preferences for AWS architecture decisions",
                "namespaces": [f"/preferences/{self.actor_id}"]
            }
        }],
        event_expiry_days=7,
    )
```

> **`MemoryClient(region_name=None, integration_source=None, boto3_session=None)`** — `bedrock_agentcore.memory`
> - 설명: AgentCore Memory 리소스를 다루는 고수준 클라이언트를 생성한다.
> - 파라미터: `region_name`(`str`, 선택) — AWS 리전, 미지정 시 세션의 리전 또는 `us-west-2`. `boto3_session`(`boto3.Session`, 선택) — 프로파일/자격 증명을 지정하기 위한 커스텀 세션.
>
> **`.create_memory_and_wait(name, strategies, description=None, event_expiry_days=90, memory_execution_role_arn=None, stream_delivery_resources=None, max_wait=300, poll_interval=10, indexed_keys=None)`**
> - 설명: Memory 리소스를 생성하고 `ACTIVE` 상태가 될 때까지 폴링하며 대기한다.
> - 파라미터:
>   - `name`(`str`, 필수): Memory 리소스 이름.
>   - `strategies`(`list[dict]`, 필수): 장기 기억 전략 설정 목록(예: `userPreferenceMemoryStrategy`).
>   - `event_expiry_days`(`int`, 기본값 `90`): 단기 기억(이벤트) 보존 기간(일).
>   - `max_wait`/`poll_interval`(`int`): 최대 대기 시간과 상태 확인 간격(초).
> - 반환값: `ACTIVE` 상태의 Memory 객체(`dict`).
> - 예외: `max_wait` 내에 `ACTIVE`가 되지 않으면 `TimeoutError`, 생성 실패 시 `RuntimeError`.

견적을 낼 때마다 `create_event`로 단기 기억에 저장하고(`estimate` 도구), `list_events`로 최근 견적들을 불러와 비교하며(`compare` 도구), `retrieve_memories`로 장기 기억을 의미론적으로 검색해 사용자 선호를 반영한 제안을 만든다(`propose` 도구).

```python
# estimate: 대화를 단기 기억에 저장
self.memory_client.create_event(
    memory_id=self.memory_id,
    actor_id=self.actor_id,
    session_id=self.session_id,
    messages=[
        (architecture_description, "USER"),
        (result, "ASSISTANT")
    ]
)

# compare: 단기 기억에서 최근 이벤트 검색
events = self.memory_client.list_events(
    memory_id=self.memory_id,
    actor_id=self.actor_id,
    session_id=self.session_id,
    max_results=4
)

# propose: 장기 기억에 대한 의미론적 검색
memories = self.memory_client.retrieve_memories(
    memory_id=self.memory_id,
    namespace=f"/preferences/{self.actor_id}",
    query=f"User preferences and decision patterns for: {requirements}",
    top_k=3
)
```

> **`.create_event(memory_id, actor_id, session_id, messages, event_timestamp=None, branch=None, metadata=None, extraction_mode=None)`**
> - 설명: 에이전트-사용자 상호작용 1건을 단기 기억 이벤트로 저장한다. 이 이벤트들이 장기 기억 추출의 원천이 된다.
> - 파라미터: `memory_id`/`actor_id`/`session_id`(`str`, 필수) — 각각 Memory 리소스, 사용자(또는 에이전트), 세션 식별자. `messages`(`list[tuple[str, str]]`, 필수) — `(텍스트, 역할)` 튜플 목록(역할은 `USER`/`ASSISTANT`/`TOOL` 등). `branch`(`dict`, 선택) — 대화 분기 정보.
> - 반환값: 생성된 이벤트(`dict`).
>
> **`.list_events(memory_id, actor_id, session_id, branch_name=None, include_parent_branches=False, event_metadata=None, max_results=100, include_payload=True)`**
> - 설명: 세션 내 단기 기억 이벤트들을 시간순으로 조회한다.
> - 파라미터: `max_results`(`int`, 기본값 `100`) — 최대 반환 개수.
> - 반환값: 이벤트 목록(`list[dict]`).
>
> **`.retrieve_memories(memory_id, namespace=None, query=None, actor_id=None, top_k=3, namespace_path=None, metadata_filters=None)`**
> - 설명: 장기 기억에 대해 쿼리와 의미론적으로 유사한 기억을 검색한다. `namespace`와 `namespace_path` 중 정확히 하나만 지정해야 한다.
> - 파라미터: `namespace`(`str`) — 정확히 일치하는 네임스페이스(예: `/preferences/{actor_id}`). `query`(`str`, 필수) — 검색 쿼리. `top_k`(`int`, 기본값 `3`) — 반환할 최대 결과 수.
> - 반환값: 관련성 순으로 정렬된 기억 레코드 목록(`list[dict]`).

## Lab 4: Observability — 모니터링 및 디버깅 📊

**목표**: AgentCore Observability로 Lab 2에서 배포한 에이전트가 요청을 받아 응답을 반환하기까지의 전체 흐름을 추적한다.

사용 사례가 단일 응답을 주는 "AI 어시스턴트"에서 자율적으로 여러 단계를 수행하는 "AI 에이전트"로 발전할수록 관찰 가능성의 중요성이 커진다(Langfuse 같은 전문 오픈소스 솔루션이 등장하는 이유). AgentCore Observability는 [[Amazon CloudWatch]]와 원클릭으로 통합되며, 소스 코드 수정 없이 활성화만으로 상세한 추적을 얻을 수 있다. 수집된 텔레메트리는 OpenTelemetry(OTEL) 형식으로 출력돼 CloudWatch 외의 APM·로그 분석 도구와도 연동 가능하다.

관찰 가능성은 **Session → Trace → Span** 3계층 구조로 제공된다. **Session**은 사용자-에이전트 간 전체 상호작용(같은 세션 ID로 여러 요청을 보내면 세션은 1개), **Trace**는 세션 내 개별 요청-응답 흐름(타임스탬프·입력·처리 단계·리소스 사용량·오류·최종 응답 포함, 3번 요청하면 Trace 3개), **Span**은 Trace 내 개별 작업(API 호출, 내부 처리 등 계층적 트리 구조)이다. 사전 조건으로 CloudWatch Transaction Search를 활성화해야 하며, 활성화 후에는 CloudWatch GenAI Observability 화면에서 Sessions·Traces·Spans를 드릴다운하며 각 단계의 모델 호출·도구 호출 세부 내용(Events 뷰)이나 처리 흐름 다이어그램(Trajectory 뷰)까지 확인할 수 있다.

```bash
# Transaction Search 활성화 확인
aws xray get-trace-segment-destination
# → {"Destination": "CloudWatchLogs", "Status": "ACTIVE"}
```

같은 세션 ID로 여러 번 `invoke_agent_runtime`을 호출하면 하나의 Session 아래 여러 Trace가 쌓이는 것을 CloudWatch에서 확인할 수 있다.

```python
response = self.client.invoke_agent_runtime(
    agentRuntimeArn=self.agent_arn,
    runtimeSessionId=session_id,
    payload=json.dumps(payload).encode('utf-8')
)
```

> **`invoke_agent_runtime(agentRuntimeArn, runtimeSessionId, payload, qualifier=None, contentType=None, accept=None, ...)`** — boto3 `bedrock-agentcore`(데이터 플레인) API
> - 설명: 배포된 AgentCore Runtime에 요청을 보낸다. 같은 `runtimeSessionId`로 여러 번 호출하면 CloudWatch Observability 상에서 하나의 Session 아래 여러 Trace로 기록된다.
> - 파라미터:
>   - `agentRuntimeArn`(`str`, 필수): 호출할 Runtime의 ARN(또는 에이전트 ID).
>   - `runtimeSessionId`(`str`): 세션 식별자.
>   - `payload`(`bytes`, 필수): 요청 본문(보통 JSON을 UTF-8로 인코딩).
>   - `qualifier`(`str`, 선택): 특정 버전을 가리키는 엔드포인트 이름.
> - 반환값: Runtime의 응답(`dict`).

## Lab 5: Evaluation — 중요한 지표 측정하기 📏

**목표**: 로컬 평가(strands-agents-evals), 온디맨드 평가(AgentCore Evaluate API), 온라인 평가(AgentCore Runtime 위의 지속적 모니터링) 세 가지 방식으로 에이전트 품질을 측정한다.

평가는 개발 아주 초기부터 시작하는 것이 바람직하다 — 측정 가능한 목표가 없으면 팀은 끝없이 프롬프트만 조정하게 되기 때문이다("평가 우선" 사고방식). 비용 추정 에이전트는 품질·비용·속도, 즉 **QCD**로 평가하는 것이 적절하다: 견적 정확도도 중요하지만 그 때문에 비용이나 대기 시간이 늘어나면 안 되므로, 가격 조회 도구 호출이 최소한으로 이뤄지는지(도구 호출)와 응답이 구체적인 비용을 담고 있는지(출력)를 평가 대상으로 삼는다.

AgentCore Evaluations는 Session/Trace/Tool 세 레벨에서 실행되는 13종의 빌트인 평가기(Helpfulness, ToolSelection, Correctness 등)를 제공하며, 커스텀 프롬프트나 독자적인 평가기 구축도 가능하다. 온디맨드 평가는 특정 Span/Trace/Session을 대상으로 한 수시 평가(디버그·문제 분석 용도), 온라인 평가는 배포된 에이전트의 라이브 트래픽에 대한 지속적 모니터링이다. 단, AgentCore Evaluations는 (2026년 2월 기준) LLM-as-a-Judge만 지원해 로컬처럼 임의의 코드를 실행하는 평가는 불가능하다. 그래서 로컬에서는 Span을 직접 검사하는 코드 기반 `ToolCallEvaluator`(커스텀)를, AgentCore 쪽에서는 도구 호출을 포함하는 Trace 레벨의 LLM-as-a-Judge 평가기를 사용하는 식으로 나눠 쓴다.

`Experiment`(테스트 케이스 + 태스크 함수 + 평가기)로 실행을 오케스트레이션하며, 빌트인 `OutputEvaluator`는 루브릭(프롬프트) 기반으로 출력을 채점하고 커스텀 `ToolCallEvaluator`는 OTel Span을 순회해 필수 도구(`get_pricing`) 호출 여부를 검사한다.

```python
from strands_evals import Case, Experiment

# 1. 테스트 케이스 정의
cases = [
    Case(
        name="single-ec2",
        input="One EC2 t3.micro instance running 24/7 in us-east-1",
        expected_trajectory=["get_pricing"],
    ),
]

# 2. 태스크 함수 정의
def task_fn(case):
    agent = AWSCostEstimatorAgent()
    output = agent.estimate_costs(case.input)
    return {"output": output, "trajectory": spans}

# 3. 평가기 정의 (루브릭 기반 OutputEvaluator + 커스텀 ToolCallEvaluator)
evaluators = [output_evaluator, tool_evaluator]

# 4. 실행
experiment = Experiment(cases=cases, evaluators=evaluators)
reports = experiment.run_evaluations(task_fn)
```

> **`Case(name=None, input, expected_output=None, expected_assertion=None, expected_trajectory=None, expected_interactions=None, metadata=None, ...)`** — `strands_evals`
> - 설명: Experiment를 구성하는 단일 테스트 케이스(Pydantic 모델).
> - 파라미터: `input`(`InputT`, 필수) — 태스크에 전달할 입력. `name`(`str`, 선택) — 리포트에 표시될 테스트 이름. `expected_trajectory`(`list`, 선택) — 기대하는 도구 호출 시퀀스. `expected_output`(`OutputT`, 선택) — 기대하는 출력.
> - 반환값: `Case` 인스턴스.
>
> **`Experiment(cases=None, evaluators=None, diagnosis_config=None)`**
> - 설명: 테스트 케이스 목록과 평가기 목록을 묶어 평가를 오케스트레이션하는 객체.
> - 파라미터: `cases`(`list[Case]`) — 실행할 테스트 케이스 목록. `evaluators`(`list[Evaluator]`) — 각 케이스에 적용할 평가기 목록.
>
> **`.run_evaluations(task, evaluation_data_store=None)`**
> - 설명: 모든 테스트 케이스에 대해 `task`를 실행하고, 모든 평가기로 채점한다(내부적으로 `run_evaluations_async`를 `max_workers=1`로 순차 실행).
> - 파라미터: `task`(`Callable[[Case], OutputT | dict]`, 필수) — 케이스를 입력받아 출력(또는 `{"output":..., "trajectory":...}`)을 반환하는 함수.
> - 반환값: 모든 (케이스, 평가기) 결과를 담은 `EvaluationReport`.

```python
from strands_evals.evaluators.evaluator import Evaluator

class ToolCallEvaluator(Evaluator[str, str]):
    def evaluate(self, evaluation_case):
        for span in evaluation_case.actual_trajectory:
            attrs = span.attributes or {}
            if attrs.get("gen_ai.operation.name") == "execute_tool":
                tool_name = attrs.get("gen_ai.tool.name", "")
                # ... required_tools와 대조
```

> **`Evaluator[InputT, OutputT]`** (베이스 클래스) — `strands_evals.evaluators.evaluator`
> - 설명: 커스텀 평가기를 만들 때 상속하는 베이스 클래스. 서브클래스는 `evaluate()`를 구현해야 한다.
> - `__init__(trace_extractor=None, name=None)`: `name`(`str`, 선택) — 평가기 인스턴스 식별자(리포트의 `evaluator` 태그로 쓰임).
> - **`.evaluate(evaluation_case)`**: 서브클래스가 오버라이드하는 추상 메서드. `evaluation_case`(`EvaluationData`) — 실제/기대 트래젝토리·출력을 담은 평가 대상.
> - 반환값: `list[EvaluationOutput]`.

온디맨드 평가는 `boto3`의 `bedrock-agentcore-control` 클라이언트로 LLM-as-a-Judge 평가기를 생성해 특정 Trace에 대해 호출하고, 온라인 평가는 배포된 Runtime을 실제로 호출하며 결과를 CloudWatch에서 확인한다(각 테스트 케이스당 약 25분 소요).

```python
import boto3

control = boto3.client("bedrock-agentcore-control")
resp = control.create_evaluator(
    evaluatorName="cost_estimator_tool_usage",
    level="TRACE",
    evaluatorConfig={
        "llmAsAJudge": {
            "instructions": "Did the agent call a pricing tool before producing costs?",
            "ratingScale": {
                "numerical": [
                    {"value": 0, "label": "No", "definition": "No pricing tool was called"},
                    {"value": 1, "label": "Yes", "definition": "Pricing tool was used"},
                ]
            },
        }
    },
)
```

> **`create_evaluator(evaluatorName, level, evaluatorConfig, description=None, kmsKeyArn=None, tags=None, clientToken=None)`** — boto3 `bedrock-agentcore-control` API
> - 설명: AgentCore 온디맨드/온라인 평가에 쓰이는 평가기(주로 LLM-as-a-Judge)를 생성한다.
> - 파라미터:
>   - `evaluatorName`(`str`, 필수): 계정 내 고유한 평가기 이름.
>   - `level`(`str`, 필수): 평가 범위 — `TOOL_CALL`(개별 도구 호출) / `TRACE`(요청-응답 흐름) / `SESSION`.
>   - `evaluatorConfig`(`dict`, 필수): LLM-as-a-Judge 설정(지침, 평점 척도) 또는 코드 기반 설정.
> - 반환값: 생성된 평가기 정보(`dict`).

## Lab 6: Identity — AI 에이전트를 위한 인증 및 권한 부여 🛡️

**목표**: AgentCore Identity로 Lab 2에서 배포한 에이전트를 누가 사용할 수 있는지 제어한다.

AgentCore Runtime은 기본적으로 IAM SigV4 인증을 요구하지만, 웹 서비스처럼 사용자 단위 인가(OAuth)가 필요하거나 사내 기존 인증·인가 체계와 연동하려면 AgentCore Identity가 필요하다. **워크로드 아이덴티티**로 "어떤 에이전트"가 IAM 역할이나 OAuth 권한·API 키에 접근할 수 있는지 일관되게 관리하며, 사용자 상호작용 기반 권한 부여 외에 예약 실행에 적합한 M2M(Machine-to-Machine) 인증도 지원한다. `@requires_access_token` 데코레이터만 추가하면 OAuth 인가 흐름을 실행하고 액세스 토큰을 얻을 수 있고, GitHub·Google·Microsoft·Slack 등 다양한 자격 증명 공급자와 쉽게 통합된다. 인증·인가 정보는 암호화돼 Token Vault에 중앙 집중 관리되며 자동으로 갱신된다.

구현 흐름은 다음과 같다: ① `GatewayClient.create_oauth_authorizer_with_cognito`로 인가 서버(Amazon Cognito)를 구축 → ② `create_oauth2_credential_provider`로 Cognito와 연동되는 AgentCore Identity(OAuth 제공자)를 구축 → ③ `authorizer_config`(discovery URL, client ID)를 지정해 권한 부여 없이는 호출할 수 없는 AgentCore Runtime을 구축.

```python
from bedrock_agentcore_starter_toolkit.operations.gateway.client import GatewayClient

gateway_client = GatewayClient(region_name=region)
cognito_result = gateway_client.create_oauth_authorizer_with_cognito("InboundAuthorizerForCostEstimatorAgent")
user_pool_id = cognito_result['client_info']['user_pool_id']
discovery_url = f"https://cognito-idp.{region}.amazonaws.com/{user_pool_id}/.well-known/openid-configuration"
cognito_config = {
    "client_id": cognito_result['client_info']['client_id'],
    "client_secret": cognito_result['client_info']['client_secret'],
    "discovery_url": discovery_url,
    "scope": cognito_result['client_info']['scope'],
}
```

> **`GatewayClient(region_name=None, endpoint_url=None)`** — `bedrock_agentcore_starter_toolkit.operations.gateway.client`
> - 설명: AgentCore Gateway 관련 작업을 위한 고수준 클라이언트.
> - 파라미터: `region_name`(`str`, 기본값 `us-west-2`).
>
> **`.create_oauth_authorizer_with_cognito(gateway_name)`**
> - 설명: Amazon Cognito User Pool·도메인·리소스 서버를 자동 생성해 OAuth 인가 서버로 구성한다.
> - 파라미터: `gateway_name`(`str`, 필수) — Cognito 리소스 이름에 쓰일 접두사.
> - 반환값: `client_info`(client_id/secret 등)를 포함한 `dict`.

```python
# AgentCore Identity에 OAuth 제공자 등록
oauth2_config = {
    'customOauth2ProviderConfig': {
        'clientId': cognito_config['client_id'],
        'clientSecret': cognito_config['client_secret'],
        'oauthDiscovery': {'discoveryUrl': cognito_config['discovery_url']}
    }
}
identity_client.create_oauth2_credential_provider(
    name=provider_name,
    credentialProviderVendor='CustomOauth2',
    oauth2ProviderConfigInput=oauth2_config
)

# authorizer_config를 지정해 인가 없이는 호출 불가능한 Runtime 생성
authorizer_config = {
    "customJWTAuthorizer": {
        "discoveryUrl": config["cognito"]["discovery_url"],
        "allowedClients": [config["cognito"]["client_id"]]
    }
}
deploy_client.create_agent_runtime(
    agentRuntimeName=secure_agent_name,
    agentRuntimeArtifact={"containerConfiguration": {"containerUri": f"{ecr_repository}:latest"}},
    roleArn=execution_role,
    authorizerConfiguration=authorizer_config
)
```

> **`create_oauth2_credential_provider(name, credentialProviderVendor, oauth2ProviderConfigInput, tags=None)`** — boto3 `bedrock-agentcore-control` API
> - 설명: AgentCore Identity에 OAuth2 자격 증명 공급자(예: Cognito, GitHub, Google)를 등록한다.
> - 파라미터: `name`(`str`, 필수) — 공급자 이름. `credentialProviderVendor`(`str`, 필수) — 공급자 종류(예: `CustomOauth2`). `oauth2ProviderConfigInput`(`dict`, 필수) — client ID/secret, discovery URL 등 공급자별 설정.
>
> **`create_agent_runtime(agentRuntimeName, agentRuntimeArtifact, roleArn, networkConfiguration, authorizerConfiguration=None, ...)`** — boto3 `bedrock-agentcore-control` API
> - 설명: AgentCore Runtime을 생성한다(콘솔의 "Agent Runtime" 항목에 대응).
> - 파라미터: `agentRuntimeName`(`str`, 필수) — Runtime 이름. `agentRuntimeArtifact`(`dict`, 필수) — 컨테이너 이미지 등 배포 아티팩트 정보. `roleArn`(`str`, 필수) — 실행 IAM 역할. `authorizerConfiguration`(`dict`, 선택) — `customJWTAuthorizer` 등 인가 설정을 지정하면 유효한 토큰 없는 요청이 거부된다.

클라이언트 측에서는 `@requires_access_token(auth_flow="M2M")`으로 액세스 토큰을 얻고, 이를 `Authorization: Bearer` 헤더에 담아 Runtime에 요청을 보낸다. 이렇게 하면 인증되지 않은 요청은 콘솔에서도 거부되고, 올바른 토큰을 가진 요청만 정상 처리된다.

```python
@requires_access_token(
    provider_name=OAUTH_PROVIDER,
    scopes=[OAUTH_SCOPE],
    auth_flow="M2M",
    force_authentication=False
)
async def _cost_estimator_with_auth(architecture_description: str, access_token: str = None) -> str:
    """Internal function that handles the actual API call with authentication"""
    if access_token:
        logger.info("✅ Successfully load the access token from AgentCore Identity!")

    headers = {
        "Authorization": f"Bearer {access_token}",
        "Content-Type": "application/json",
    }
    response = requests.post(RUNTIME_URL, headers=headers,
                              data=json.dumps({"prompt": architecture_description}))
    response.raise_for_status()
    return response.text
```

> **`@requires_access_token(*, provider_name, into="access_token", scopes, resources=None, audiences=None, on_auth_url=None, auth_flow, callback_url=None, force_authentication=False, token_poller=None, custom_state=None, custom_parameters=None)`** — `bedrock_agentcore.identity.auth`
> - 설명: 데코레이트된 함수를 호출하기 전에 AgentCore Identity를 통해 OAuth2 액세스 토큰을 발급받아 함수 인자로 주입한다.
> - 파라미터:
>   - `provider_name`(`str`, 필수): 사용할 자격 증명 공급자 이름.
>   - `scopes`(`list[str]`, 필수): 요청할 OAuth2 스코프.
>   - `auth_flow`(`"M2M" | "USER_FEDERATION" | "ON_BEHALF_OF_TOKEN_EXCHANGE"`, 필수): 인증 흐름 종류. `M2M`은 사용자 개입 없는 서버 간 인증.
>   - `into`(`str`, 기본값 `"access_token"`): 토큰을 주입할 함수 파라미터 이름.
>   - `force_authentication`(`bool`, 기본값 `False`): 캐시된 토큰이 있어도 재인증을 강제할지 여부.
> - 반환값: 원본 함수를 감싼 래퍼(동기/비동기 모두 지원).

## Lab 7: Gateway — 외부 서비스를 MCP로 변환하고 연결하기 🔌

**목표**: AgentCore Gateway로 외부 도구(이메일 전송)를 MCP를 통해 사용할 수 있게 하고, Lab 6에서 만든 Cognito 인가 서버로 Gateway 호출을 보호한다.

기업 내외부에는 이미 다양한 API(OpenAPI, Smithy, AWS Lambda 등)가 존재하는데, AgentCore Gateway는 이런 API를 MCP 형식으로 변환해 에이전트가 곧바로 활용할 수 있게 해준다. Salesforce·Slack·Jira·Asana·Zendesk 같은 주요 서비스는 원클릭 통합도 제공한다. 인바운드(에이전트→Gateway) 인증에는 AgentCore Identity의 OAuth를, 아웃바운드(Gateway→외부 서비스) 인증에는 별도의 권한 부여 방식을 각각 설정할 수 있어 포괄적인 인증 체계를 이룬다. 수천 개의 도구가 등록돼도 의미론적 검색으로 필요한 도구만 최소한의 프롬프트로 호출할 수 있고, 서버리스라 인프라 관리가 필요 없으며 Observability도 내장돼 있다.

실습에서는 Markdown 형식의 비용 추정 보고서를 HTML로 변환해 Amazon SES로 발송하는 AWS Lambda(`src/app.py`)를 SAM으로 배포한 뒤, `create_mcp_gateway`(인바운드: Lab 6의 Cognito 설정 재사용)와 `create_gateway_target`(아웃바운드: Lambda ARN + `GATEWAY_IAM_ROLE`)으로 Gateway를 구축한다.

```python
# 인바운드: Lab 6의 Cognito 인가 서버 재사용
gateway = gateway_client.create_mcp_gateway(
    name="AWSCostEstimatorGateway",
    role_arn=None,
    authorizer_config={
        "customJWTAuthorizer": {
            "discoveryUrl": identity_config["cognito"]["discovery_url"],
            "allowedClients": [identity_config["cognito"]["client_id"]]
        }
    },
    enable_semantic_search=False
)

# 아웃바운드: Lambda를 MCP 대상으로 등록
control_client.create_gateway_target(
    gatewayIdentifier=gateway_id,
    name="AWSCostEstimatorGatewayTarget",
    targetConfiguration={
        "mcp": {"lambda": {"lambdaArn": config["lambda_arn"], "toolSchema": {"inlinePayload": tool_schema}}}
    },
    credentialProviderConfigurations=[{"credentialProviderType": "GATEWAY_IAM_ROLE"}]
)
```

> **`.create_mcp_gateway(name=None, role_arn=None, authorizer_config=None, enable_semantic_search=True, enable_observability=True, policy_engine_config=None)`** — `GatewayClient`
> - 설명: MCP 프로토콜을 쓰는 AgentCore Gateway를 생성한다(역할·인가 서버가 없으면 자동 생성, CloudWatch Observability 기본 활성화).
> - 파라미터: `authorizer_config`(`dict`, 선택) — 인바운드 인가 설정, 미지정 시 Cognito 인가 서버를 새로 만든다. `enable_semantic_search`(`bool`, 기본값 `True`) — 도구가 많을 때 의미론적 검색으로 필요한 도구만 찾도록 지원.
> - 반환값: 생성된 Gateway 정보(`dict`, `gatewayArn`/`gatewayUrl` 포함).
>
> **`create_gateway_target(gatewayIdentifier, name, targetConfiguration, credentialProviderConfigurations=None, description=None, metadataConfiguration=None, privateEndpoint=None, clientToken=None)`** — boto3 `bedrock-agentcore-control` API
> - 설명: Gateway에 외부 서비스(Lambda, OpenAPI 등)를 MCP 도구로 등록한다(아웃바운드 연결 대상).
> - 파라미터: `gatewayIdentifier`(`str`, 필수) — 대상 Gateway 식별자. `targetConfiguration`(`dict`, 필수) — 대상 엔드포인트·스키마 설정(예: `{"mcp": {"lambda": {...}}}`). `credentialProviderConfigurations`(`list`, 선택) — 아웃바운드 인증 방식(예: `GATEWAY_IAM_ROLE`).

클라이언트는 `@requires_access_token`으로 얻은 토큰을 MCP Client(`streamablehttp_client`)에 담아 Gateway에 접속하고, `list_tools_sync`로 Gateway에 등록된 도구(이메일 전송)와 비용 추정 도구를 함께 에이전트에 등록해 "견적 계산 → 이메일 발송"까지 한 번에 처리한다.

```python
def create_transport():
    return streamablehttp_client(GATEWAY_URL, headers={"Authorization": f"Bearer {access_token}"})

mcp_client = MCPClient(create_transport)

tools = [cost_estimator_tool]
with mcp_client:
    tools.extend(mcp_client.list_tools_sync())  # Gateway에 등록된 markdown_to_email 등 포함

    agent = Agent(
        system_prompt=(
            "1. Please summarize customer's requirement to `architecture_description`."
            "2. Pass `architecture_description` to 'cost_estimator_tool'."
            "3. Send estimation by `markdown_to_email`."
        ),
        tools=tools
    )
```

> **`streamablehttp_client(url, headers=None, timeout=30, sse_read_timeout=300, auth=None)`** — `mcp.client.streamable_http`(MCP Python SDK)
> - 설명: Streamable HTTP 방식으로 MCP 서버(Gateway)에 연결하는 트랜스포트를 여는 비동기 컨텍스트 매니저.
> - 파라미터: `url`(`str`, 필수) — MCP 서버 엔드포인트. `headers`(`dict`, 선택) — 요청에 포함할 HTTP 헤더(여기서는 `Authorization: Bearer <token>`).
> - 반환값: `(read_stream, write_stream, get_session_id)` 튜플.
> - 참고: 이 이름은 MCP SDK v1 기준이며, 워크샵은 이 버전을 사용한다. v2부터는 이 함수가 제거되고 `streamable_http_client`(파라미터로 헤더 대신 `httpx.AsyncClient` 사용)로 대체됐다.
>
> **`MCPClient(transport_callable, *, startup_timeout=30, tool_filters=None, prefix=None, continue_on_error=False, ...)`** — Strands Agents SDK (`strands.tools.mcp`)
> - 설명: MCP 서버와의 연결을 관리하며 서버가 제공하는 도구를 Strands `Agent`가 쓸 수 있는 `AgentTool`로 변환한다.
> - 파라미터: `transport_callable`(`Callable[[], MCPTransport]`, 필수) — 트랜스포트를 반환하는 콜러블(예: `streamablehttp_client`를 감싼 함수). `startup_timeout`(`int`, 기본값 `30`) — 서버 초기화 타임아웃(초).
>
> **`.list_tools_sync(pagination_token=None, prefix=None, tool_filters=None)`**
> - 설명: MCP 서버에 등록된 도구 목록을 동기적으로 조회해 `AgentTool` 목록으로 반환한다.
> - 반환값: `PaginatedList[MCPAgentTool]`.

## Lab 8: Policy — 세분화된 도구 액세스 제어 🔐

**목표**: AgentCore Policy로 Lab 7에서 만든 이메일 전송 도구를 특정 역할(Manager)만 쓸 수 있도록 제한한다.

인증된 사용자는 기본적으로 모든 도구를 사용할 수 있는데, IAM은 AWS 서비스 레벨에서 작동하기 때문에 "도구 레벨"의 제어(예: Developer는 내부 검토용 견적만 작성하고, 클라이언트에게 실제 이메일을 보내는 것은 Manager만 가능해야 하는 상황)에는 대응하지 못한다. AgentCore Policy는 오픈소스 **Cedar** 언어 기반 정책으로 이 문제를 해결한다. 정책 적용은 AI 에이전트의 비결정론적 동작과 독립적으로, 사전 정의된 규칙에 따라 결정론적으로 이뤄지므로 프레임워크·모델에 관계없이 일관되게 허용/거부를 판단할 수 있다. 자연어에서 Cedar 정책을 생성하는 **NL2Cedar** 기능도 제공해 새로운 문법을 배우는 학습 비용을 줄여주고, 과도하게 허용적이거나 제한적인 규칙을 자동 추론으로 감지한다. 정책 적용 결과는 CloudWatch에 기록돼 감사·모니터링이 가능하며, 실제 적용(ENFORCE) 전에 로그만 남기는 모드로 미리 테스트할 수도 있다.

Cedar 정책은 `permit(principal, action, resource) when { 조건 }` 형태로 작성한다(반대는 `forbid`이며 `forbid`가 항상 `permit`을 재정의). 기본 동작은 전체 거부이며, 일치하는 `permit`이 없으면 도구 호출이 차단된다.

```cedar
permit (                           -- 효과: permit 또는 forbid
  principal is <PrincipalType>,    -- 누가 요청하는가?
  action == <Action>,              -- 어떤 도구/작업을 호출하는가?
  resource == <Resource>           -- 어떤 Gateway를 대상으로 하는가?
)
when {                             -- 조건: 추가 조건(선택 사항)
  <조건식>
};
```

이번 실습에서는 JWT 토큰의 `scope` 클레임에 `manager`가 포함된 경우에만 `markdown_to_email` 도구 호출을 허용하는 정책을 만들어(`setup_policy.py`), Developer 역할로 호출하면 이메일 도구 자체가 도구 목록에 나타나지 않고(기본 거부), Manager 역할로 호출하면 정상적으로 이메일이 발송되는 것을 확인한다.

```cedar
permit(
  principal,
  action == AgentCore::Action::"AWSCostEstimatorGatewayTarget___markdown_to_email",
  resource == AgentCore::Gateway::"arn:aws:bedrock-agentcore:...:gateway/..."
) when {
  principal.hasTag("scope") &&
  principal.getTag("scope") like "*manager*"
};
```

Cedar 문법을 직접 익히지 않아도, 자연어로 원하는 규칙을 설명하면 **NL2Cedar**가 정책을 생성해준다.

```python
nl_description = (
    "Allow any user whose OAuth token scope contains 'manager' "
    "to use the markdown_to_email tool on the gateway. "
    "Deny all other users from using the markdown_to_email tool."
)

generation = policy_client.generate_policy(
    policy_engine_id=engine_id,
    name="scope_based_email_policy",
    resource={"arn": gateway_arn},
    content={"rawText": nl_description},
    fetch_assets=True,
)
```

> **`.generate_policy(policy_engine_id, name, resource, content, client_token=None, max_attempts=30, delay=2, fetch_assets=False)`** — `PolicyClient`(Starter Toolkit), 내부적으로 boto3 `StartPolicyGeneration` 오퍼레이션 사용
> - 설명: 자연어 설명(NL2Cedar)으로부터 Cedar 정책을 생성하고, 생성이 완료(`GENERATED`)될 때까지 폴링한 뒤 결과를 반환하는 편의 메서드.
> - 파라미터:
>   - `policy_engine_id`(`str`, 필수): 대상 Policy Engine ID.
>   - `resource`(`dict`, 필수): 정책 적용 대상 리소스(예: `{"arn": gateway_arn}`).
>   - `content`(`dict`, 필수): 자연어 설명(예: `{"rawText": "..."}`).
>   - `fetch_assets`(`bool`, 기본값 `False`): `True`면 생성된 Cedar 정책 텍스트까지 함께 가져온다.
> - 반환값: 생성 결과(`fetch_assets=True`면 `generatedPolicies` 필드 포함).
> - 예외: 제한 시간 내 완료되지 않으면 `TimeoutError`.

## Lab 9: Browser Use — 웹 폼 자동화 🌐

**목표**: AgentCore Browser로 API가 없는 웹 폼을 조작하는 방법을 익힌다.

기업 내 프로젝트 관리 도구, 티켓 포털, 오래된 [[온프레미스]] 시스템 등은 API를 제공하지 않는 경우가 많다. HTML 폼만 있는 도구에 에이전트가 만든 비용 추정 결과를 입력하고 싶을 때 API도 MCP도 쓸 수 없는데, 이때 AgentCore Browser가 해결책이 된다. 완전 관리형 원격 브라우저 환경으로 인프라 관리 없이 수천 개의 동시 세션을 병렬 실행할 수 있고(기본 타임아웃 15분, 최대 8시간), WebSocket 기반 스트리밍 API로 탐색·클릭·입력·스크린샷 등을 수행하며 Playwright 같은 라이브러리와 연동할 수 있다. 세션 녹화는 S3에 저장돼 재생할 수 있고, 콘솔에서 라이브 모니터링도 가능하다.

구현은 `browser_session` 컨텍스트 매니저로 AgentCore Browser 세션을 시작하고, `generate_ws_headers`로 얻은 WebSocket URL·SigV4 인증 헤더를 이용해 Playwright의 `connect_over_cdp`(Chrome DevTools Protocol)로 연결하는 방식이다.

```python
from bedrock_agentcore.tools.browser_client import browser_session
from playwright.sync_api import sync_playwright

with browser_session(region) as client:
    ws_url, headers = client.generate_ws_headers()

    with sync_playwright() as pw:
        browser = pw.chromium.connect_over_cdp(ws_url, headers=headers)
        page = browser.contexts[0].pages[0]
        page.goto("https://example.com")
```

> **`browser_session(region, viewport=None, identifier=None, name=None, proxy_configuration=None, extensions=None, profile_configuration=None, enterprise_policies=None, certificates=None)`** — `bedrock_agentcore.tools.browser_client`
> - 설명: AgentCore Browser 샌드박스 세션을 생성·시작하고 종료까지 관리하는 컨텍스트 매니저.
> - 파라미터: `region`(`str`, 필수)만 있으면 시스템 기본 브라우저로 즉시 사용 가능. 나머지는 프록시·확장 프로그램 등 커스터마이징 옵션.
> - 반환값(yield): 시작된 `BrowserClient` 인스턴스.
>
> **`.generate_ws_headers()`**
> - 설명: 브라우저 스트림에 연결할 WebSocket URL과 SigV4 서명이 포함된 인증 헤더를 생성한다.
> - 반환값: `(ws_url, headers)` 튜플.
>
> **`chromium.connect_over_cdp(endpoint_url, headers=None, slow_mo=None, timeout=30000, ...)`** — Playwright Python(`BrowserType.connect_over_cdp`)
> - 설명: Chrome DevTools Protocol로 기존(원격) 브라우저 인스턴스에 연결한다.
> - 파라미터: `endpoint_url`(`str`, 필수) — CDP WebSocket 또는 HTTP 엔드포인트. `headers`(`dict`, 선택) — 연결 요청에 포함할 HTTP 헤더. `timeout`(`float`, 기본값 `30000`) — 연결 대기 시간(ms).
> - 반환값: `Browser` 인스턴스.

폼 필드는 하드코딩하지 않고, `page.locator("body").aria_snapshot()`으로 접근성 트리를 텍스트로 캡처한 뒤 이를 견적 결과와 함께 Bedrock에 전달해 필드명-값 매핑(`{"textboxes": {...}, "radios": {...}}`)을 생성하는 방식으로 동적으로 발견한다. 이 방식은 대상 폼의 레이아웃이 바뀌거나 다른 폼이어도 그대로 적용할 수 있다는 장점이 있다.

```python
# 1. 접근성 트리를 텍스트로 캡처
form_snapshot = page.locator("body").aria_snapshot()

# 2. 스냅샷과 추정을 Bedrock에 전송하여 필드 값 매핑 획득
form_values = generate_form_values(estimation_text, form_snapshot, region)
# → {"textboxes": {"Title of target system": "Simple Web Application",
#                   "Total monthly cost": "$137.44", ...},
#    "radios": {"Amazon EC2": true}}
```

> **`Locator.aria_snapshot(*, boxes=False, depth=None, mode="default", timeout=30000)`** — Playwright Python
> - 설명: 요소와 그 하위 트리의 접근성(ARIA) 상태를 YAML 형식 문자열로 캡처한다. 폼 필드를 하드코딩하지 않고 동적으로 발견하는 데 사용된다.
> - 파라미터: `depth`(`int`, 선택) — 스냅샷 깊이 제한. `timeout`(`float`, 기본값 `30000`) — 최대 대기 시간(ms).
> - 반환값: ARIA 스냅샷을 담은 `str`.

## Lab A-1: Kiro로 커스텀 에이전트 개발

**목표**: 지금까지 배운 내용을 활용해 AWS AI 코딩 에이전트 도구 **Kiro**로 자신만의 에이전트를 만들고 AgentCore에 배포한다.

Kiro는 "Spec Driven Development"를 지원하는 통합 개발 환경이다. AI 코딩 에이전트를 프롬프트로만 다루면 프롬프트에 담긴 정보량(컨텍스트)에 따라 코드 품질이 크게 흔들리고, 입력한 프롬프트가 문서로 남지 않아 사양이 공유되지 않는 문제가 있었다. Kiro는 `requirements.md`(사양) → `design.md`(설계) → `task.md`(구현 작업)를 GUI 기반으로 문서화하며 진행하게 해 이 문제를 해결한다. Spec Driven이 필수는 아니며, 커맨드라인 도구 `kiro-cli`로 가볍게 프롬프트만 주고받을 수도 있고, 익숙한 VS Code 환경에서는 Amazon Q Developer Extension으로도 AI 에이전트를 호출할 수 있다.

실습에서는 AWS Builder ID로 `kiro-cli login --use-device-flow`(디바이스 플로우로 브라우저 인증) 로그인 후, `.kiro/agents`에 정의된 `agent-builder` 에이전트로 Kiro CLI를 실행한다.

```bash
kiro-cli login --use-device-flow
cd a1_custom
kiro-cli --agent agent-builder
```

Lab 2의 Runtime 배포 코드를 참고해 원하는 기능(예: 날씨 예보를 반환하는 에이전트)의 사양을 `README.md`에 먼저 작성하도록 요청하고, 내용을 검토·수정한 뒤 "README.md대로 구현해줘"라고 요청하면 구현이 진행된다. 배포는 Lab 2와 동일하게 `agentcore launch --local-build`로 수행하며, 오류가 나면 오류 메시지를 Kiro CLI에 그대로 붙여넣어 위치를 특정하고 수정을 요청할 수 있다.

```
당신은 신속한 프로토타이핑에 능숙한 개발자입니다.
02-runtime 폴더의 내용을 기반으로 Amazon Bedrock AgentCore에 배포 가능한 에이전트를 a1_custom 폴더 내에 구현합니다.
구현에 앞서 폴더 내 README.md에 다음 기능을 가진 에이전트의 사양, 설계, 구현에 필요한 작업 3가지를 기재하세요.

* 사용자의 질문에 대해 날씨 예보를 반환하는 에이전트
```

```
README.md대로 구현해줘
```

---

## 정리

9개 랩을 거치며 보안 코드 실행(Code Interpreter) → 클라우드 배포(Runtime) → 메모리(Memory) → 관찰 가능성(Observability) → 평가(Evaluation) → 인증(Identity) → 외부 통합(Gateway) → 세분화된 액세스 제어(Policy) → 브라우저 자동화(Browser Use) 순으로 점진적으로 고급 에이전트 기능을 쌓아 올렸다. AgentCore의 핵심 가치는 두 가지로 요약된다.

- **높은 효율성으로 고급 에이전트 개발**: 확장성·보안을 오프로드해 에이전트 핵심 로직 구현에 집중할 수 있고, 필요한 기능만 골라 채택할 수 있으며, microVM 세션 격리·엔드투엔드 추적 등 프로덕션에 필요한 기능을 기본 제공한다.
- **기술 부채 방지**: 특정 프레임워크를 가정하지 않는 컨테이너 기반 배포와 기능 통합으로, 인증·관찰성처럼 에이전트 주변에서 방치되기 쉬운 기능들이 레거시로 굳어지는 것을 막는다.

이제 막 AI 에이전트 개발을 시작한다면 **Strands Agents**로 빠르게 프로토타이핑하고, 이미 에이전트를 운영 중이라면 AgentCore Runtime(보안·확장성)·Identity(인가)·Observability(모니터링)를 순서대로 도입하는 경로가 권장된다. 실습 후 리소스는 각 랩 디렉토리의 `clean_resources.py`(또는 Gateway의 경우 `sam delete`)로 랩 9→8→7→6→5→3→2 역순으로 정리하고, 사전 준비 단계에서 만든 개발 환경 자체를 더 이상 쓰지 않는다면 CloudFormation 스택도 삭제한다.

```bash
# 1. Clean up Browser Use sessions
cd 09_browser_use
uv run python clean_resources.py

# 2. Clean up Policy resources
cd 08_policy
uv run python clean_resources.py

# 3. Clean up Gateway resources (uses SAM CLI)
cd 07_gateway
sam delete  # Lambda 함수 및 관련 리소스 삭제
uv run python clean_resources.py

# 4. Clean up Identity resources
cd 06_identity
uv run python clean_resources.py

# 5. Clean up Evaluation resources
cd 05_evaluation
uv run python clean_resources.py

# 6. Clean up Memory resources
cd 03_memory
uv run python clean_resources.py

# 7. Clean up Runtime resources
cd 02_runtime
uv run python clean_resources.py
```

## 관련 노트

- [[AgentCoreWithWebApp 빈칸 채우기 실습]] — 이 워크샵의 Lab 1·2·3·6·7 코드를 하나의 채팅 웹앱으로 통합해 빈칸 채우기로 만든 후속 실습
- [[Developing Generative AI Applications on AWS]] — 같은 교육에서 AgentCore를 Module 10으로 소개한 상위 강의
- [[AI Infra 교육 정리본]] — 이 워크샵(5일차)을 포함한 SWM AI Infra 교육 5일 전체 정리본
- [[서버리스]] — AgentCore Runtime이 채택한, 요청 처리 시에만 과금되는 실행 모델
- [[온프레미스]] — Lab 9 Browser Use가 대응하는, API가 없는 레거시 온프레미스 시스템
- [[Amazon CloudWatch]] — Lab 4 Observability의 Session·Trace·Span 추적과 Lab 8 Policy 감사 로그가 기록되는 관측 서비스
