---
title: 'Claude Managed Agents — 멀티 에이전트 세션'
date: 2026-07-14 14:13:09 +0900
categories: [개발, AI]
tags: [claude, multi-agent, managed-agents, orchestration, mcp, anthropic, api]
description: '코디네이터 에이전트가 다른 에이전트에 작업을 위임해 병렬로 처리하는 Claude Managed Agents API의 멀티 에이전트 세션 오케스트레이션 정리.'
---
Claude Managed Agents API의 멀티 에이전트 오케스트레이션은 코디네이터 에이전트가 다른 에이전트들에게 일을 위임해 복잡한 작업을 처리하게 해준다. 각 에이전트는 격리된 컨텍스트를 갖고 병렬로 동작하므로 결과 품질과 완료 시간이 모두 개선될 수 있다.

## 동작 방식

- 모든 에이전트는 같은 샌드박스·파일시스템·볼트(vault) 자격증명을 공유하지만, 각자 자기만의 **세션 스레드**(대화 이력이 분리된 컨텍스트 격리 이벤트 스트림)에서 실행된다.
- 코디네이터는 **프라이머리 스레드**(세션 수준 이벤트 스트림과 동일)에 활동을 보고하고, 작업을 위임할 때 추가 스레드가 런타임에 생성된다.
- 스레드는 영속적이다. 코디네이터가 이전에 호출한 에이전트에게 후속 메시지를 보내면, 그 에이전트는 이전 턴의 내용을 전부 기억한다.
- 각 에이전트는 모델·시스템 프롬프트·툴·MCP 서버·스킬을 자기 설정대로 쓴다. 툴, MCP 서버, 컨텍스트는 공유되지 않는다. 예외적으로 세션 수준 에이전트 설정 오버라이드는 코디네이터와 그 `self` 복사본에만 적용된다.
- Managed Agents API 요청에는 `managed-agents-2026-04-01` 베타 헤더가 필요하다(메모리 스토어 엔드포인트만 `agent-memory-2026-07-22` 사용). SDK는 올바른 베타 헤더를 자동으로 설정한다.

## 어떤 일을 위임할까

여러 표면에 걸친 작업이나, 잘 쪼개진 하위 작업 여러 개가 하나의 목표에 기여하는 복잡한 태스크에 적합하다. 잘 맞는 패턴:

- **병렬화(Parallelization)**: 독립적인 하위 작업(여러 소스 검색, 개별 파일 분석)을 동시에 뿌리고 코디네이터가 결과를 종합
- **전문화(Specialization)**: 한 에이전트에 모든 능력을 몰아넣는 대신 보안 에이전트, 문서 에이전트처럼 도메인 특화 시스템 프롬프트·툴을 가진 에이전트로 라우팅
- **에스컬레이션(Escalation)**: 복잡한 일부 하위 작업만 더 강력한 에이전트/모델에 상담

## 코디네이터 설정

에이전트를 정의할 때 `multiagent` 필드로 위임 가능한 에이전트 로스터를 선언한다.

```
ant beta:agents create <<YAML
name: Engineering Lead
model: claude-opus-4-8
system: You coordinate engineering work. Delegate code review to the reviewer agent and test writing to the test agent.
tools:
  - type: agent_toolset_20260401
multiagent:
  type: coordinator
  agents:
    - type: agent
      id: $REVIEWER_AGENT_ID
    - type: agent
      id: $TEST_WRITER_AGENT_ID
YAML
```

`multiagent.agents`에 넣을 수 있는 항목:

- `{"type": "agent", "id": agent.id}` — 기존 에이전트를 ID로 참조. `version` 미지정 시 코디네이터 생성 시점의 최신 버전에 고정(pin)된다.
- `{"type": "agent", "id": agent.id, "version": agent.version}` — 특정 버전 고정
- `{"type": "self"}` — 코디네이터가 자기 자신의 복사본을 생성 가능. 세션 생성 시의 설정 오버라이드가 이 복사본들에도 적용된다.

주의할 제약:

- 코디네이터 설정(로스터 포함)은 생성/업데이트 시점에 스냅샷된다. 참조된 에이전트가 나중에 업데이트돼도 자동 반영되지 않으므로, 새 버전에 위임하려면 코디네이터를 업데이트해야 한다.
- 위임은 1단계 깊이만 가능하다(depth > 1은 무시됨).
- `multiagent.agents`에는 최대 20개의 고유 에이전트를 등록할 수 있고, 각 에이전트의 복사본은 여러 개 호출할 수 있다.

세션은 코디네이터를 참조해 생성한다.

```
session = client.beta.sessions.create(
    agent=coordinator.id,
    environment_id=environment.id,
)
```

## MCP 서버 연결

MCP 서버는 **에이전트 스코프**(각 에이전트 정의가 자기 서버·툴을 선언)이고, 볼트 자격증명은 **세션 스코프**(세션 생성 시 넘긴 `vault_ids`가 모든 스레드에 적용)다. 따라서:

- 모든 에이전트가 쓰는 MCP 서버마다 볼트 자격증명을 포함해야 인증이 된다.
- 에이전트의 접근을 제한하려면 해당 에이전트 정의에 필요한 서버만 선언한다.

예시에서는 researcher 에이전트만 GitHub MCP 서버를 선언하므로 코디네이터는 접근 권한이 없고, 세션의 `vault_ids`가 researcher 스레드에 GitHub 자격증명을 공급한다. 서버 선언 후에도 MCP 인증이 실패하면 자격증명의 `mcp_server_url`이 에이전트의 `mcp_servers[].url`과 스킴·트레일링 슬래시까지 정확히 일치하는지 확인한다.

## 스레드와 이벤트

- 세션 수준 이벤트 스트림(`/v1/sessions/:id/events/stream`)이 **프라이머리 스레드**로, 모든 스레드 활동의 요약 뷰를 담는다. 서브에이전트의 전체 활동은 보이지 않지만 작업 시작/종료와 툴 권한 요청 같은 블로킹 이벤트는 보인다.
- 특정 에이전트의 활동을 파고들려면 해당 **세션 스레드**의 이벤트를 스트리밍하거나 목록으로 조회한다.
- 세션 `status`는 전체 에이전트 활동의 집계다. 스레드 하나라도 `running`이면 세션도 `running`.
- 동시 스레드는 최대 25개까지 지원된다.
- 스레드 목록에서 프라이머리 스레드는 `parent_thread_id`가 null이다.

프라이머리 스레드에 표면화되는 멀티 에이전트 이벤트:

| 타입 | 설명 |
| --- | --- |
| `session.thread_created` | 스레드 생성됨 (`session_thread_id`, `agent_name` 포함) |
| `session.thread_status_running` | 스레드가 활동 시작 |
| `session.thread_status_idle` | 에이전트가 입력 대기 중 (`stop_reason` 포함) |
| `session.thread_status_terminated` | 스레드가 아카이브되거나 종료 오류 발생 |
| `agent.thread_message_received` | 에이전트가 코디네이터에게 결과 전달 |
| `agent.thread_message_sent` | 코디네이터가 다른 에이전트에게 후속 메시지 전송 |

## 툴 권한과 커스텀 툴

서브에이전트가 클라이언트의 응답을 필요로 하면(`always_ask` 툴 권한, 커스텀 툴 결과 등) 해당 이벤트가 원 스레드를 가리키는 `session_thread_id`와 함께 **프라이머리 스레드로 크로스 포스팅**된다. 클라이언트는 `user.tool_confirmation`(`tool_use_id` 포함) 또는 `user.custom_tool_result`(`custom_tool_use_id` 포함)를 보내면 되고, 서버가 응답을 올바른 스레드로 자동 라우팅한다.

## 관련 노트

- [learn-claude-code-bash-is-all-you-need](/posts/learn-claude-code-bash-is-all-you-need/) — 서브에이전트 격리·컨텍스트 관리 같은 멀티에이전트 하네스 메커니즘을 0부터 구현하며 설명
- [sakana-fugu](/posts/sakana-fugu/) — 멀티에이전트 오케스트레이션을 API로 다루는 또 다른 사례
- [art-of-loop-engineering](/posts/art-of-loop-engineering/) — 코디네이터가 에이전트를 반복 호출하는 에이전트 루프의 설계 관점

> 원문: [Multi-agent sessions](https://platform.claude.com/docs/en/managed-agents/multi-agent)
> 원본 클립: 2026-07-14-Multi-agent sessions
