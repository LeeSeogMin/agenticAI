# 7장 보충 리서치: Google ADK, A2A 실전, 로우코드 에이전트 빌더, OpenAI/AutoGen 업데이트

## 조사일: 2026-02-11

---

## 1. Google ADK (Agent Development Kit)

### 1.1 출시 및 버전 정보

- **최초 출시**: 2025년 4월 9일, Google Cloud Next 2025에서 공개
- **TypeScript 버전**: 2025년 11월 7일 출시 (오픈소스)
- **최신 버전**: v1.23.0 (2026-01-22 릴리스), 약 2주 간격 릴리스 주기
- **지원 언어**: Python (v0.1.0+), TypeScript (v0.2.0+), Go (v0.1.0+), Java (v0.1.0+, 현재 v0.5.0)
- **GitHub Stars**: 약 17.2k (adk-python, 2026년 2월 기준)
- **라이선스**: Apache 2.0 (오픈소스)
- **PyPI 패키지명**: `google-adk`

**출처:**
- https://github.com/google/adk-python
- https://github.com/google/adk-python/releases/tag/v1.23.0
- https://google.github.io/adk-docs/release-notes/
- https://pypi.org/project/google-adk/
- https://developers.googleblog.com/introducing-agent-development-kit-for-typescript-build-ai-agents-with-the-power-of-a-code-first-approach/

### 1.2 핵심 에이전트 유형

ADK는 세 가지 에이전트 범주를 제공한다.

#### (1) LLM Agent (LlmAgent)
- 대형 언어 모델로 구동되는 에이전트
- 지시문(instructions), 도구(tools), 하위 에이전트(sub_agents) 설정 가능
- `output_key` 속성으로 최종 응답을 세션 상태에 자동 저장

#### (2) Workflow Agent (워크플로우 에이전트)
실행 흐름을 관리하는 전문 에이전트 세 종류:

**SequentialAgent (순차 에이전트)**
- 조립 라인처럼 하위 에이전트를 순서대로 실행
- 동일한 InvocationContext를 순차 전달하여 공유 상태(shared state)를 통한 결과 전달 가능
- 용도: 파이프라인 처리, 리뷰/비평 패턴

**ParallelAgent (병렬 에이전트)**
- 모든 하위 에이전트를 동시에(concurrently) 실행
- 독립적 작업을 병렬 처리하여 성능 향상
- 모든 병렬 자식 에이전트가 동일한 session.state에 접근 가능
- 용도: 팬아웃/수집(Fan-Out/Gather) 패턴

**LoopAgent (반복 에이전트)**
- 하위 에이전트를 반복 순차 실행
- 종료 조건: `max_iterations` 도달 또는 하위 에이전트가 `escalate=True` 이벤트 반환
- 용도: 반복 개선(Iterative Refinement) 패턴

#### (3) Custom Agent (커스텀 에이전트)
- BaseAgent를 상속하여 비-LLM 전문 로직 구현

```python
# ADK 멀티에이전트 구성 예시
from google.adk.agents import SequentialAgent, ParallelAgent, LoopAgent, LlmAgent

# 개별 에이전트 정의
researcher = LlmAgent(name="researcher", instructions="...", output_key="research_result")
writer = LlmAgent(name="writer", instructions="...", output_key="draft")
reviewer = LlmAgent(name="reviewer", instructions="...", output_key="review")

# 순차 파이프라인 구성
pipeline = SequentialAgent(
    name="content_pipeline",
    sub_agents=[researcher, writer, reviewer]
)

# 병렬 실행 구성
parallel_research = ParallelAgent(
    name="parallel_research",
    sub_agents=[web_searcher, db_searcher, api_caller]
)

# 반복 개선 루프
refinement_loop = LoopAgent(
    name="refinement",
    sub_agents=[writer, reviewer],
    max_iterations=3
)
```

### 1.3 계층적 멀티에이전트 구조

- 부모-자식 관계의 트리 구조로 에이전트 구성
- `sub_agents` 매개변수에 에이전트 인스턴스 리스트를 전달하면 프레임워크가 자동으로 `parent_agent` 속성 설정
- **제약**: 하나의 에이전트 인스턴스는 한 번만 하위 에이전트로 추가 가능 (중복 시 ValueError)

**에이전트 간 상호작용 메커니즘 3가지:**

| 메커니즘 | 설명 | 용도 |
|---------|------|------|
| Shared Session State | `context.state['key']`로 데이터 읽기/쓰기 | 워크플로우 에이전트 간 데이터 전달 |
| LLM-Driven Delegation | `transfer_to_agent(agent_name='...')` 함수 호출 | 동적 에이전트 라우팅 |
| AgentTool | 대상 에이전트를 호출 가능한 도구로 래핑 | 명시적 호출, 결과 반환 |

### 1.4 ADK vs LangGraph vs OpenAI Agents SDK 비교

| 비교 항목 | Google ADK | LangGraph | OpenAI Agents SDK |
|----------|-----------|-----------|-------------------|
| **설계 철학** | 코드 퍼스트, 배터리 포함 | 명시적 그래프 기반 | 최소주의, 핸드오프 중심 |
| **오케스트레이션** | 이벤트 기반 워크플로우 | 노드/엣지 그래프 | 핸드오프 기반 위임 |
| **멀티에이전트** | Sequential/Parallel/Loop + 계층 | StateGraph + 조건부 엣지 | Agent 배열 + Handoff |
| **상태 관리** | 공유 세션 상태 | Checkpointer + 명시적 상태 | RunContext |
| **모델 종속성** | Gemini 최적화, 타 모델 호환 | 모델 무관 | OpenAI 모델 전용 |
| **생태계** | Google Cloud, Vertex AI 통합 | LangChain 에코시스템 | OpenAI API 통합 |
| **GitHub Stars** | ~17.2k | ~22.9k | ~18.4k |
| **커뮤니티 성숙도** | 신생 (2025.4~) | 성숙 | 중간 (2025.3~) |
| **성능** | 중간 | 최저 지연시간, 최소 토큰 사용 | 중간 |
| **배포** | Cloud Run, 로컬, 컨테이너 | 어디서든 | OpenAI 플랫폼 |

**선택 가이드라인:**
- **ADK**: Google Cloud 생태계, 계층적 에이전트 팀, "배터리 포함" 개발 경험 선호 시
- **LangGraph**: 정밀한 워크플로우 제어, 명시적 상태 관리, 커스텀 사이클 그래프 필요 시
- **OpenAI Agents SDK**: 빠른 프로토타이핑, OpenAI 모델 중심 프로젝트, 단순한 핸드오프 패턴 시

**출처:**
- https://google.github.io/adk-docs/agents/multi-agents/
- https://developers.googleblog.com/developers-guide-to-multi-agent-patterns-in-adk/
- https://www.zenml.io/blog/google-adk-vs-langgraph
- https://medium.com/google-cloud/langgraph-vs-adk-a-developers-guide-to-choosing-the-right-ai-agent-framework-b59f756bcd98

---

## 2. A2A 프로토콜 실전 구현

### 2.1 A2A 프로토콜 개요

- **공개**: 2025년 4월 9일, Google Cloud Next 2025
- **현재 버전**: v0.3 (gRPC 지원, 에이전트 카드 서명, Python SDK 클라이언트 확장)
- **거버넌스**: Linux Foundation 기증, Apache 2.0 라이선스
- **지원 기업**: 50개 이상 파트너 (Atlassian, Salesforce, PayPal 등)
- **기반 기술**: HTTP, SSE (Server-Sent Events), JSON-RPC 2.0
- **공식 사이트**: https://a2a-protocol.org

**버전 히스토리:**
- v0.1: 초기 릴리스
- v0.2: 무상태(stateless) 상호작용, OpenAPI 기반 인증 스키마 표준화
- v0.3: gRPC 지원, 에이전트 카드 서명, 확장된 클라이언트 SDK

### 2.2 에이전트 카드(Agent Card) 작성법

에이전트 카드는 `.well-known/agent.json` (또는 `.well-known/agent-card.json`) 경로에 게시되는 JSON 메타데이터 문서이다. 에이전트의 "디지털 명함"으로 기능하여 검색과 상호작용 설정을 가능하게 한다.

#### 필수 필드 (Required)

| 필드 | 타입 | 설명 |
|-----|------|------|
| `name` | string | 에이전트 이름 |
| `description` | string | 에이전트 목적/기능 설명 |
| `version` | string | 에이전트 구현 버전 |
| `url` | string | A2A 서비스 엔드포인트 URL |
| `capabilities` | AgentCapabilities | 기능 지원 선언 (streaming, pushNotifications 등) |
| `defaultInputModes` | string[] | 기본 입력 미디어 타입 |
| `defaultOutputModes` | string[] | 기본 출력 미디어 타입 |

#### 선택 필드 (Optional)

| 필드 | 타입 | 설명 |
|-----|------|------|
| `skills` | AgentSkill[] | 에이전트가 제공하는 기술 목록 |
| `extensions` | AgentExtension[] | 커스텀 확장 기능 |
| `signature` | AgentCardSignature | 암호학적 서명 검증 |
| `securitySchemes` | SecurityScheme[] | 인증 방법 (API Key, OAuth2, mTLS 등) |
| `tags` | string[] | 분류 태그 |

#### 에이전트 카드 JSON 예시

```json
{
  "name": "Currency Exchange Agent",
  "description": "실시간 환율 조회 및 환전 계산을 수행하는 에이전트",
  "version": "1.0.0",
  "url": "https://api.example.com/a2a",
  "defaultInputModes": ["text"],
  "defaultOutputModes": ["text"],
  "capabilities": {
    "streaming": true,
    "pushNotifications": false,
    "extendedAgentCard": true
  },
  "skills": [
    {
      "id": "currency_convert",
      "name": "환율 변환",
      "description": "두 통화 간 실시간 환율을 조회하고 변환 금액을 계산한다",
      "tags": ["finance", "currency", "exchange"],
      "examples": [
        "1000달러를 원화로 변환해줘",
        "EUR to KRW exchange rate"
      ],
      "inputModes": ["text"],
      "outputModes": ["text"]
    }
  ],
  "securitySchemes": [
    {
      "type": "apiKey",
      "in": "header",
      "name": "X-API-Key"
    }
  ]
}
```

#### Python SDK로 에이전트 카드 정의

```python
from a2a.types import AgentCard, AgentSkill, AgentCapabilities

skill = AgentSkill(
    id='currency_convert',
    name='환율 변환',
    description='두 통화 간 실시간 환율을 조회하고 변환 금액을 계산한다',
    tags=['finance', 'currency'],
    examples=['1000달러를 원화로 변환해줘'],
)

agent_card = AgentCard(
    name='Currency Exchange Agent',
    description='실시간 환율 조회 및 환전 계산 에이전트',
    url='http://localhost:9999/',
    version='1.0.0',
    default_input_modes=['text'],
    default_output_modes=['text'],
    capabilities=AgentCapabilities(streaming=True),
    skills=[skill],
)
```

### 2.3 클라이언트/리모트 에이전트 태스크 위임 흐름

A2A에서 태스크 위임은 다음 생명주기를 따른다:

```
[1] 클라이언트 → 리모트: SendMessage(message, configuration)
        │
[2] 리모트 → 클라이언트: Task 객체 반환 (taskId, contextId 포함)
        │                또는 Message 객체 직접 반환
[3] 태스크 상태 추적 (3가지 방식)
    ├── 폴링(Polling): GetTask(taskId) 주기적 호출
    ├── 스트리밍(Streaming): SendStreamingMessage() 또는 SubscribeToTask()
    └── 푸시 알림(Push): 에이전트가 클라이언트 웹훅으로 POST
        │
[4] 종료 상태
    ├── completed (성공)
    ├── failed (오류)
    ├── canceled (사용자 취소)
    ├── rejected (유효성 검증 실패)
    └── input_required (추가 입력 필요 → 멀티턴)
```

**태스크 상태 전이:**
`submitted` → `working` → `completed` | `failed` | `canceled` | `input_required`

**멀티턴 대화:** `input_required` 상태에서 동일한 `taskId` + `contextId`로 후속 메시지를 보내면 컨텍스트가 유지된다.

### 2.4 크로스 프레임워크 에이전트 연동

A2A의 핵심 가치는 서로 다른 프레임워크로 구축된 에이전트가 동일한 프로토콜로 통신할 수 있다는 점이다.

#### 실전 예시: 구매 대리인 시스템

Google Codelab에서 제공하는 실전 예시:
- **구매 대리인 (Google ADK + Ollama)**: 대화를 오케스트레이션
- **버거 판매 에이전트 (CrewAI + vLLM)**: 버거 주문 처리
- **피자 판매 에이전트 (LangGraph + Ollama)**: 피자 주문 처리

루트 에이전트가 A2A 프로토콜을 통해 판매 에이전트에게 작업을 위임하여, 프레임워크 경계를 넘은 원활한 협업을 구현한다.

#### A2A + MCP + ADK 통합 패턴

```
[에이전트 개발] ──── ADK (Agent Development Kit)
       │
[도구 연동] ──────── MCP (Model Context Protocol)
       │                 에이전트 ↔ 외부 도구/서비스
[에이전트 통신] ──── A2A (Agent-to-Agent Protocol)
                       에이전트 ↔ 에이전트
```

- **ADK**: 에이전트 개발 및 관리
- **MCP**: 에이전트가 외부 도구/서비스에 접근하는 표준
- **A2A**: 에이전트 간 통신 및 태스크 위임 표준

이 세 기술은 경쟁이 아니라 보완 관계이다.

#### A2A 어댑터 SDK

`hybroai/a2a-adapter` (GitHub)는 다양한 에이전트 프레임워크 (n8n, LangGraph, CrewAI, LangChain 등)를 A2A 프로토콜과 통합하는 Python SDK를 제공한다.

**출처:**
- https://a2a-protocol.org/latest/specification/
- https://a2a-protocol.org/latest/tutorials/python/3-agent-skills-and-card/
- https://codelabs.developers.google.com/intro-a2a-purchasing-concierge
- https://codelabs.developers.google.com/codelabs/currency-agent
- https://github.com/a2aproject/A2A
- https://github.com/hybroai/a2a-adapter
- https://cloud.google.com/blog/products/ai-machine-learning/agent2agent-protocol-is-getting-an-upgrade
- https://heemeng.medium.com/playing-around-with-a2a-langgraph-crewai-0f47d9414eb6

---

## 3. 로우코드/노코드 에이전트 빌더

### 3.1 n8n

#### 기본 정보
- **GitHub Stars**: ~173.8k (2026년 2월 기준)
- **라이선스**: Fair-code (소스 공개, 상업적 제한 조건 있음)
- **기술 스택**: Node.js (TypeScript), 최근 네이티브 Python 실행 지원 추가
- **통합**: 400개 이상 사전 구축 통합
- **커뮤니티**: 200k+ 회원
- **GitHub**: https://github.com/n8n-io/n8n

#### AI 에이전트 핵심 기능

1. **내장 AI 에이전트 빌더**: 메모리, 도구, 가드레일을 갖춘 컨텍스트 인식 에이전트를 시각적으로 설계
2. **MCP 통합**: Model Context Protocol 지원으로 AI 모델이 도구를 동적으로 교체 가능
3. **메모리 노드**:
   - Redis Chat Memory (대화 기록)
   - Postgres/Supabase Vector Memory (RAG용)
   - Simple Window Memory (멀티턴 대화)
4. **네이티브 Python 실행**: Node.js 외에 Pandas, NumPy 등 데이터 과학 라이브러리 직접 실행 지원
5. **에이전트 결정 로직**: 사용자 의도에 따라 n8n 노드를 자동 트리거
6. **셀프 호스팅**: 데이터 주권 보장, 온프레미스 배포 지원

#### 강점
- 범용 워크플로우 자동화 + AI 에이전트 통합
- 수백 개 사전 빌트 통합으로 외부 서비스 연결 용이
- 팀 협업 지원 (다양한 기술 수준의 팀원 참여 가능)
- 빠른 프로토타이핑에 강점

### 3.2 Dify

#### 기본 정보
- **GitHub Stars**: ~129k (2026년 2월 기준, 100k 돌파 시 세계 상위 100 오픈소스 프로젝트)
- **라이선스**: Apache 2.0 (오픈소스)
- **최초 공개**: 2023년 5월 15일
- **기술 스택**: TypeScript (프론트엔드/백엔드)
- **GitHub**: https://github.com/langgenius/dify

#### 핵심 기능

1. **LLM 오케스트레이션**: 다양한 언어 모델을 원활하게 연결/전환
2. **비주얼 스튜디오**: 드래그 앤 드롭 방식의 AI 워크플로우 설계
3. **배포 허브**: AI 애플리케이션을 API, 챗봇, 내부 비즈니스 도구로 원클릭 배포
4. **에이전틱 워크플로우**: 프로덕션 레디 에이전트 워크플로우 개발 플랫폼
5. **RAG 파이프라인**: 기본 내장 RAG 기능
6. **플러그인 통합**: 확장 가능한 플러그인 아키텍처

### 3.3 n8n vs Dify 비교

| 비교 항목 | n8n | Dify |
|----------|-----|------|
| **핵심 정체성** | 범용 워크플로우 자동화 플랫폼 | AI 네이티브 앱 개발 플랫폼 |
| **비유** | "신경계" (입출력 연결) | "두뇌" (로직, 추론, 의사결정) |
| **주요 용도** | IT 자동화, 시스템 통합, 에이전트 오케스트레이션 | LLM 앱 구축, 평가, 배포 |
| **AI 에이전트** | 워크플로우 내 AI 에이전트 노드 | 에이전트 중심 설계 |
| **통합 수** | 400+ | 상대적으로 적음 (LLM 중심) |
| **GitHub Stars** | ~173.8k | ~129k |
| **라이선스** | Fair-code | Apache 2.0 |
| **기술 스택** | Node.js + Python | TypeScript |
| **상호 보완** | Dify API를 n8n에서 호출 가능 | n8n 웹훅으로 Dify 트리거 가능 |

**핵심 차이**: Dify는 AI 앱 개발에 특화된 플랫폼이고, n8n은 범용 자동화에 AI 에이전트 기능을 추가한 플랫폼이다. 두 도구는 API/웹훅을 통해 연동하여 "AI 추론 → 기업 액션"의 완전한 루프를 구성할 수 있다.

### 3.4 기타 주요 로우코드 에이전트 도구

#### Flowise
- **GitHub Stars**: ~48.9k
- **기술 스택**: Node.js
- **특징**: LLM 파이프라인(프롬프트 체인, 검색, 도구, 에이전트) 시각적 구성
- **강점**: RAG 중심 앱, 빠른 피드백 루프, 셀프 호스팅 + 커스터마이징 유연성
- **GitHub**: https://github.com/FlowiseAI/Flowise

#### Langflow
- **GitHub Stars**: ~144.6k (DataStax 인수로 재정적 안정성 확보)
- **기술 스택**: Python (LangChain 기반)
- **특징**: 간결함과 속도에 강점, 사전 제작 템플릿, 빠른 챗봇/문서 QA 구축
- **강점**: 초보자 친화적, Python 기반으로 소스 코드 접근 가능
- **GitHub**: https://github.com/langflow-ai/langflow

### 3.5 시각적 도구 vs 코드 기반 프레임워크 선택 기준

#### 시각적/로우코드 도구 적합 상황
- 엔지니어링 리소스가 제한된 팀
- 복잡한 내부 도구 통합 및 PoC (Proof of Concept)
- 다양한 기술 수준의 팀원이 참여하는 협업
- 오케스트레이션, 관측성, 스케줄링, 커넥터가 에이전트 내부 로직보다 중요한 경우
- 빠른 프로토타이핑 후 코드로 전환하는 하이브리드 전략

#### 코드 기반 프레임워크 적합 상황
- 프로덕션 규모의 애플리케이션
- 고속, 세밀한 제어, 극한의 유연성 필요 시
- 계획(planning), 메모리, 도구 사용, 멀티에이전트 협업에 대한 정밀 제어 필요
- 전문 로직이나 틈새 통합이 필요한 경우
- 규정 준수, 감사, 보안 요구사항이 높은 경우

#### 하이브리드 접근법
많은 팀이 n8n으로 빠른 프로토타이핑을 시작하고, 이후 Python 코드 기반 프레임워크 (LangGraph, ADK 등)로 전환하여 스케일링과 고급 커스터마이징을 수행한다. 핵심은 "미래 대비"보다 "현재 제약 조건 (규정 준수, 팀 기술, 배포 요구)"에 기반한 선택이다.

**출처:**
- https://github.com/n8n-io/n8n
- https://github.com/langgenius/dify
- https://github.com/FlowiseAI/Flowise
- https://n8n.io/ai-agents/
- https://dify.ai/blog/100k-stars-on-github-thank-you-to-our-amazing-open-source-community
- https://hostadvice.com/blog/ai/automation/n8n-vs-dify/
- https://www.langflow.org/blog/the-complete-guide-to-choosing-an-ai-agent-framework-in-2025
- https://inkeep.com/blog/agent-frameworks-platforms-overview
- https://flowiseai.com/

---

## 4. OpenAI Agents SDK 핸드오프 및 AutoGen v0.4/AG2 업데이트

### 4.1 OpenAI Agents SDK 핸드오프 메커니즘

#### 핸드오프 개요
핸드오프(Handoff)는 에이전트가 다른 에이전트에게 작업을 위임하는 메커니즘이다. LLM에게는 도구(tool)로 표현되며, "Refund Agent"로의 핸드오프는 `transfer_to_refund_agent` 도구가 된다.

#### 도구 네이밍 컨벤션
에이전트 이름이 소문자로 변환되고, 공백이 밑줄로 대체된다.
- "Travel Genie" → `transfer_to_travel_genie`
- "Billing Agent" → `transfer_to_billing_agent`

#### 핸드오프 코드 예시

```python
from agents import Agent, handoff, RunContextWrapper
from pydantic import BaseModel

# 전문 에이전트 정의
billing_agent = Agent(
    name="Billing agent",
    instructions="청구 관련 문의를 처리한다..."
)

refund_agent = Agent(
    name="Refund agent",
    instructions="환불 요청을 처리한다..."
)

# 트리아지 에이전트 (라우터)
triage_agent = Agent(
    name="Triage agent",
    handoffs=[billing_agent, handoff(refund_agent)]
)
```

#### 핸드오프 커스터마이징 옵션

| 매개변수 | 설명 |
|---------|------|
| `agent` | 위임 대상 에이전트 |
| `tool_name_override` | 도구 이름 커스텀 (기본: `transfer_to_<name>`) |
| `tool_description_override` | 도구 설명 커스텀 |
| `on_handoff` | 핸드오프 실행 시 콜백 함수 |
| `input_type` | 핸드오프 시 전달할 Pydantic 모델 |
| `input_filter` | 다음 에이전트에 전달할 대화 히스토리 필터링 |
| `is_enabled` | 런타임 핸드오프 활성화/비활성화 (bool 또는 함수) |

#### 고급 기능: 에스컬레이션 데이터 전달

```python
class EscalationData(BaseModel):
    reason: str

async def on_handoff(ctx: RunContextWrapper[None], input_data: EscalationData):
    print(f"에스컬레이션 사유: {input_data.reason}")

handoff_obj = handoff(
    agent=escalation_agent,
    on_handoff=on_handoff,
    input_type=EscalationData,
)
```

#### 입력 필터

`agents.extensions.handoff_filters`에서 사전 구축된 필터 제공:
- `remove_all_tools`: 도구 호출 이력을 대화 히스토리에서 제거

#### 핸드오프 프롬프트

```python
from agents.extensions.handoff_prompt import RECOMMENDED_PROMPT_PREFIX

agent = Agent(
    name="Billing agent",
    instructions=f"""{RECOMMENDED_PROMPT_PREFIX}
    청구 관련 문의를 처리한다...""",
)
```

#### 네스트 핸드오프 (Beta)
`RunConfig.nest_handoff_history`를 활성화하면 이전 대화 기록을 `<CONVERSATION HISTORY>` 블록으로 요약하여 전달한다.

### 4.2 AutoGen v0.4: 이벤트 기반 아키텍처

#### 릴리스 정보
- **릴리스**: 2025년 1월 (Microsoft Research)
- **GitHub**: https://github.com/microsoft/autogen

#### 핵심 아키텍처 변경

AutoGen v0.4는 비동기, 이벤트 기반, 액터 모델 기반으로 완전히 재설계되었다.

**3계층 API 구조:**

| 계층 | 용도 | 대상 |
|-----|------|------|
| **AgentChat** | 빠른 멀티에이전트 앱 구축 | 프로토타이핑 |
| **Core** | 이벤트 파이프라인, 스케일링 | 프로덕션 |
| **Extensions** | 모델/도구 통합 | 커스터마이징 |

**핵심 개선사항:**
- 비동기 이벤트 기반 메시징 시스템
- 이벤트 기반 + 요청-응답 상호작용 패턴 모두 지원
- 확장성, 모듈성, 디버깅 도구 강화
- 크로스 언어 지원

### 4.3 Magentic-One

Microsoft Research가 개발한 범용 멀티에이전트 시스템으로, 복잡한 웹 및 파일 기반 작업 해결에 특화되었다.

#### 에이전트 구성 (5개)

| 에이전트 | 역할 |
|---------|------|
| **Orchestrator** | 작업 분해, 계획, 하위 에이전트 지시, 진행 추적, 수정 |
| **WebSurfer** | Chromium 기반 브라우저 제어 (내비게이션, 웹 페이지 액션, 읽기) |
| **FileSurfer** | 마크다운 기반 파일 프리뷰로 로컬 파일 읽기 |
| **Coder** | 코드 작성, 디버깅, 다른 에이전트 입력 해석 |
| **ComputerTerminal** | Coder가 생성한 코드를 실제 실행 |

#### 핵심 메커니즘
- **Task Ledger**: 작업 분해 및 추적
- **Progress Ledger**: 진행 상황 모니터링 및 계획 수정
- **LLM 무관**: GPT-4o 최적화이나 다른 모델 사용 가능

### 4.4 AG2 포크 현황

#### 배경
- 2024년 11월, AutoGen 원 창시자 Chi Wang과 Qingyun Wu가 Microsoft를 떠나 AG2 (AutoGen 2.0)를 커뮤니티 기반 포크로 설립
- PyPI 패키지 (`autogen`, `pyautogen`)와 Discord 커뮤니티의 통제권 유지

#### 현재 상태 (2026년 2월)
- **GitHub**: https://github.com/ag2ai/ag2 (~4.1k stars)
- **PyPI**: `ag2` (또는 `autogen` alias)
- **거버넌스**: AG2AI 조직 설립, 오픈 거버넌스 프레임워크
- **Python 요구사항**: >= 3.10, < 3.14
- **Discord**: 20k+ AI 에이전트 개발자

#### Microsoft AutoGen vs AG2 비교

| 비교 항목 | Microsoft AutoGen (v0.4) | AG2 |
|----------|------------------------|-----|
| **관리 주체** | Microsoft Research | AG2AI (커뮤니티) |
| **아키텍처** | 완전 재설계 (이벤트 기반) | AutoGen 0.2 아키텍처 유지 |
| **초점** | 엔터프라이즈 스케일, 관측성 | 커뮤니티 기반, 후방 호환성 |
| **거버넌스** | 기업 주도 | 오픈 거버넌스 |
| **패키지명** | `autogen-agentchat` (PyPI) | `ag2` / `autogen` (PyPI) |

#### 포크 이유
1. 기업 제약 없이 더 빠른 의사결정과 효율적 개발
2. 다른 대형 기업 기여를 포함한 중립적 개발 공간 조성

**출처:**
- https://openai.github.io/openai-agents-python/handoffs/
- https://openai.github.io/openai-agents-python/multi_agent/
- https://devblogs.microsoft.com/autogen/autogen-reimagined-launching-autogen-0-4/
- https://www.microsoft.com/en-us/research/blog/autogen-v0-4-reimagining-the-foundation-of-agentic-ai-for-scale-extensibility-and-robustness/
- https://microsoft.github.io/autogen/dev/user-guide/agentchat-user-guide/magentic-one.html
- https://github.com/ag2ai/ag2
- https://arxiv.org/html/2411.04468v1

---

## 부록: 프레임워크 종합 비교 (2026년 2월 기준)

| 프레임워크 | GitHub Stars | 유형 | 멀티에이전트 | 특징 |
|-----------|-------------|------|------------|------|
| n8n | ~173.8k | 로우코드 자동화 | 워크플로우 기반 | 400+ 통합, AI 에이전트 노드 |
| Langflow | ~144.6k | 로우코드 빌더 | 시각적 구성 | LangChain 기반, DataStax 인수 |
| Dify | ~129k | AI 앱 플랫폼 | 에이전틱 워크플로우 | RAG, LLM 오케스트레이션 |
| Flowise | ~48.9k | 로우코드 빌더 | 시각적 구성 | Node.js, RAG 특화 |
| LangGraph | ~22.9k | 코드 프레임워크 | 그래프 기반 | 명시적 상태, 최저 지연 |
| OpenAI Agents SDK | ~18.4k | 코드 프레임워크 | 핸드오프 기반 | OpenAI 전용, 간결 |
| Google ADK | ~17.2k | 코드 프레임워크 | 계층적 워크플로우 | Google Cloud 통합, A2A/MCP |
| AG2 | ~4.1k | 코드 프레임워크 | 대화 기반 | AutoGen 0.2 호환, 커뮤니티 |

---

**리서치 완료일**: 2026-02-11
**담당**: @researcher
