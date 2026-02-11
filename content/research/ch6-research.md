# 6장 리서치 결과: LangGraph로 상태·분기·반복을 제어한다

## 조사일: 2026-02-11 (2차 보강)

---

## 1. LangGraph 1.0 (2025년 10월 릴리스)

### 1.1 릴리스 일자 및 배경

LangGraph 1.0은 **2025년 10월 22일** 일반 공개(GA)되었다. LangChain 1.0과 동시에 발표되었으며, "에이전트 프레임워크 최초의 안정 메이저 릴리스"로 위치한다. Uber, LinkedIn, Klarna 등 기업에서 1년 이상 프로덕션 운영한 결과를 바탕으로 안정화된 릴리스다.

- 출처: [LangGraph 1.0 is now generally available](https://changelog.langchain.com/announcements/langgraph-1-0-is-now-generally-available)
- 출처: [LangChain and LangGraph Agent Frameworks Reach v1.0 Milestones](https://blog.langchain.com/langchain-langgraph-1dot0/)

### 1.2 API 안정성 보장

**v2.0까지 브레이킹 체인지 없음**을 공식 보장한다. v0.6.6에서 v1.0으로의 업그레이드에서 브레이킹 체인지가 전혀 없다. 유일한 변경점은 `langgraph.prebuilt.create_react_agent`의 폐기(deprecation)이며, 이 함수는 `langchain.agents.create_agent`으로 이전되었다. 제거(removal)는 v2.0에서 예정이다.

- 출처: [What's new in LangGraph v1](https://docs.langchain.com/oss/python/releases/langgraph-v1)
- 출처: [LangGraph 1.0: Changes Guide](https://medium.com/@romerorico.hugo/langgraph-1-0-released-no-breaking-changes-all-the-hard-won-lessons-8939d500ca7c)

### 1.3 다운로드 수

PyPI 통계 기준(2026년 2월 시점):
- **최근 30일: 약 3,026만 다운로드** (pypistats.org 기준)
- 최근 7일: 약 867만 다운로드
- 일일: 약 75만 다운로드

> **참고**: `contents.md`에 기재된 "월 617만 다운로드"는 과거 시점(추정 2025년 중반)의 수치이며, 2026년 2월 현재 월 3,000만 이상으로 대폭 증가했다. 다만 PyPI 다운로드 수치는 CI/CD 파이프라인, 미러, 봇 트래픽을 포함하므로 실제 사용자 수와는 차이가 있다.

- 출처: [PyPI Download Stats - langgraph](https://pypistats.org/packages/langgraph)
- 출처: [langgraph - PyPI](https://pypi.org/project/langgraph/)

### 1.4 핵심 기능

#### StateGraph
- LangGraph의 주요 그래프 클래스. `TypedDict` 또는 `dataclass`로 State를 정의한다
- 빌더 패턴: `StateGraph(State)` → `add_node()` → `add_edge()` / `add_conditional_edges()` → `.compile()`
- 컴파일 후 `invoke()`, `stream()`, `astream()`, `ainvoke()` 메서드로 실행

```python
from typing import TypedDict, Annotated
from operator import add

class WorkflowState(TypedDict):
    messages: list
    draft: str
    feedback: Annotated[list[str], add]  # 리스트 누적
    revision_count: int
```

#### 조건부 엣지(Conditional Edges)
- `builder.add_conditional_edges("source_node", routing_function, path_map)` 형태
- 라우팅 함수가 문자열을 반환하면, path_map에서 대응하는 노드로 분기
- 복수 분기 경로를 명시적으로 정의 가능

#### 순환(Cycles)
- DAG가 아닌 순환 그래프를 네이티브 지원 (이것이 LangGraph가 일반 DAG 프레임워크와 차별되는 핵심)
- 무한 루프 방지 권장 패턴: `max_steps` 카운터, 지수 백오프, 명시적 탈출 조건
- Pregel 및 Apache Beam에서 영감을 받은 실행 모델

#### 체크포인팅(Checkpointing)
- 모든 슈퍼스텝(superstep)마다 그래프 상태의 스냅샷을 자동 저장
- 서버 재시작이나 장기 실행 워크플로우 중단 시 정확히 중단 지점에서 재개
- 스레드(thread) 개념: 고유 thread_id로 독립적인 대화/세션 관리
- 빌트인 체크포인터: `MemorySaver`(개발용), `PostgresSaver`, `SqliteSaver`
- 서드파티: `langgraph-checkpoint-redis`, `langgraph-checkpoint-mongodb`

- 출처: [Graph API overview](https://docs.langchain.com/oss/python/langgraph/graph-api)
- 출처: [LangGraph GitHub](https://github.com/langchain-ai/langgraph)

### 1.5 v1.0에서 달라진 점 (pre-1.0 대비)

v1.0은 새 기능 추가보다 **안정성 확정**에 초점을 맞춘다. 핵심 그래프 프리미티브(state, nodes, edges)는 변경 없음.

| 항목 | pre-1.0 (0.x) | v1.0 |
|------|---------------|------|
| API 안정성 | 실험적, 수시 변경 | v2.0까지 안정 보장 |
| create_react_agent | langgraph.prebuilt에 위치 | deprecated → langchain.agents.create_agent |
| 내구 실행(Durable Execution) | 존재했으나 비공식 | 공식 지원, 프로덕션 검증 |
| 스트리밍 | 기본 지원 | LLM 토큰, 도구 호출, 상태 업데이트, 노드 전환 전체 스트리밍 |
| Human-in-the-loop | 패턴으로 존재 | 1등급 API 지원 (interrupt → resume) |
| Functional API | 미존재 | `@task`, `@entrypoint` 데코레이터 기반 함수형 API 추가 |
| 메모리 | 체크포인터 중심 | 단기 메모리(체크포인터) + 장기 메모리(Store API) 이원 체계 |

- 출처: [What's new in LangGraph v1](https://docs.langchain.com/oss/python/releases/langgraph-v1)

---

## 2. LangSmith 통합

### 2.1 LangGraph 트레이싱 연동

LangSmith는 LangGraph(Python/JS)와 네이티브 통합된다. LangChain 모듈을 LangGraph 내에서 사용하는 경우, 환경 변수 설정만으로 트레이싱이 활성화된다:

```bash
export LANGSMITH_TRACING=true
export LANGSMITH_API_KEY="ls__..."
export LANGSMITH_PROJECT="my-langgraph-project"
```

2025년 7월 31일부터, LangSmith 내에서 트레이스를 LangGraph Platform의 서버 로그와 직접 연결하는 기능이 추가되었다.

- 출처: [Trace LangGraph applications](https://docs.langchain.com/langsmith/trace-with-langgraph)
- 출처: [Connect traces in LangSmith to server logs](https://changelog.langchain.com/announcements/connect-traces-in-langsmith-to-server-logs-in-langgraph-platform)

### 2.2 중첩 스팬(Nested Span) 트레이싱

LangSmith의 트레이싱 모델:
- **Run**: 에이전트가 수행하는 개별 단계(LLM 호출, 도구 호출 등)
- **Trace**: 에이전트의 단일 실행 단위. 트리 구조의 Run들로 구성
- 부모-자식 관계로 중첩(nested) 시각화: 각 스팬을 확장하면 입력, 출력, 메타데이터, 소요 시간 확인 가능
- 10개 이상의 도구 호출이 포함된 멀티스텝 에이전트도 즉시 재생(replay) 가능
- 각 단계의 레이턴시를 시각적으로 추적하여 병목 지점 식별

커스텀 함수에 `@traceable` 데코레이터를 적용하면, 모델 호출 옆에 커스텀 로직도 트레이스에 포함된다.

- 출처: [LangSmith Observability](https://www.langchain.com/langsmith/observability)
- 출처: [Advanced LangSmith Agent Tracing Techniques](https://sparkco.ai/blog/advanced-langsmith-agent-tracing-techniques-in-2025)
- 출처: [Debugging Deep Agents with LangSmith](https://blog.langchain.com/debugging-deep-agents-with-langsmith/)

### 2.3 평가 워크플로우(Evaluation Workflows)

LangSmith는 오프라인/온라인 양면의 평가 체계를 제공한다:

**오프라인 평가**:
1. 데이터셋 생성: 수동 큐레이션, 프로덕션 트레이스, 합성 데이터로 구성
2. 에이전트를 전체 데이터셋에 대해 실행
3. 평가자(Evaluator) 적용: LLM-as-judge, 코드 기반 휴리스틱, 커스텀 로직
4. 프롬프트/모델 버전 간 비교

**평가 방식 혼합**:
- 인간 리뷰: 미묘한 판단이 필요한 경우
- 휴리스틱: 명확한 규칙 기반 검증
- LLM-as-judge: 스케일이 필요한 경우
- 버전 고정 데이터셋으로 재현성 확보

- 출처: [LangSmith Evaluation](https://docs.langchain.com/langsmith/evaluation)
- 출처: [LangSmith Evaluation Platform](https://www.langchain.com/langsmith/evaluation)

### 2.4 LangSmith 요금제

| 플랜 | 좌석 | 월 요금 | 포함 트레이스 | 보존 기간 |
|------|------|--------|-------------|----------|
| Developer (무료) | 1석 | $0 | 5,000 base traces/월 | 14일 |
| Plus | 최대 10석 | $39/석/월 | 10,000 base traces/월 | 14일 (base) |
| Enterprise | 무제한 | 커스텀 | 커스텀 | 커스텀 |

**추가 트레이스 비용**:
- Base traces (14일 보존): $2.50 / 1,000건
- Extended traces (400일 보존): $5.00 / 1,000건

Plus 플랜에는 개발용 LangGraph Platform 배포 1개가 무료 포함된다. Startup 플랜(초기 스타트업용)도 별도 존재한다.

- 출처: [LangSmith Plans and Pricing](https://www.langchain.com/pricing)
- 출처: [LangSmith Pricing FAQ](https://docs.langchain.com/langsmith/pricing-faq)

---

## 3. LangGraph + 장기 메모리(Long-term Memory)

### 3.1 체크포인팅 vs 장기 메모리의 차이

LangGraph는 메모리를 **두 가지 계층**으로 구분한다:

| 구분 | 체크포인터 (Checkpointer) | 스토어 (Store) |
|------|--------------------------|---------------|
| 범위 | 단일 스레드(대화) 내 | 스레드 간(cross-thread) |
| 용도 | 멀티턴 대화 컨텍스트, 중단 복구 | 사용자 선호, 학습된 지식, 과거 결정 |
| 데이터 | 그래프 실행 상태의 스냅샷 | JSON 문서 (key-value) |
| 수명 | 스레드 종료 시 의미 감소 | 애플리케이션 수명과 동일 |
| 검색 | thread_id + checkpoint_id | namespace + key, 또는 의미 검색(semantic search) |

체크포인터는 "이 대화에서 어디까지 진행했는가"를 다루고, 스토어는 "이 사용자에 대해 무엇을 알고 있는가"를 다룬다.

- 출처: [Memory overview - LangChain Docs](https://docs.langchain.com/oss/python/langgraph/memory)
- 출처: [The difference between Checkpointers and Storage](https://github.com/langchain-ai/langgraph/discussions/3496)

### 3.2 Store API 핵심 메서드

LangGraph의 `BaseStore` 인터페이스는 세 가지 핵심 메서드를 제공한다:

- **`put(namespace, key, value)`**: 네임스페이스에 JSON 문서를 저장/갱신
- **`get(namespace, key)`**: 네임스페이스와 키로 단일 문서 조회
- **`search(namespace, query=..., filter=...)`**: 네임스페이스 내 문서 검색. 의미 검색(semantic search) 지원 시 자연어 쿼리로 유사도 기반 검색 가능

네임스페이스는 계층적으로 구성 가능하다. 예: `("users", "user_123", "preferences")`

```python
# 메모리 저장
store.put(("users", "user_123"), "pref", {"language": "ko", "tone": "formal"})

# 메모리 조회
item = store.get(("users", "user_123"), "pref")

# 의미 검색 (semantic search 활성화 시)
results = store.search(("users", "user_123"), query="사용자가 선호하는 언어")
```

### 3.3 의미 검색(Semantic Search)

기본적으로 Store는 의미 검색 없이 동작하지만, `IndexConfig`를 설정하면 벡터 기반 검색이 활성화된다:

- 오픈소스 구현: `PostgresStore`, `InMemoryStore`에서 지원
- LangGraph Platform 프로덕션 배포에서도 지원
- 임베딩 모델, 차원, 검색 대상 필드를 설정

- 출처: [Semantic Search for LangGraph Memory](https://blog.langchain.com/semantic-search-for-langgraph-memory/)
- 출처: [Storage (LangGraph) Reference](https://reference.langchain.com/python/langgraph/store/)
- 출처: [How to add semantic search to your agent's memory](https://langchain-ai.github.io/langgraph/how-tos/memory/semantic-search/)

### 3.4 MongoDB 통합

`langgraph-store-mongodb` 패키지가 `MongoDBStore` 클래스를 제공한다:

- MongoDB의 네이티브 JSON 문서 구조와 LangGraph의 JSON 메모리 형식이 직접 매핑
- MongoDB Atlas Vector Search로 의미 기반 메모리 검색
- TTL(Time-to-Live) 인덱스로 오래된 메모리 자동 정리
- 기존 `langgraph-checkpoint-mongodb`(체크포인터)와 병행하여 단기/장기 메모리 완전 커버

- 출처: [Powering Long-Term Memory For Agents With LangGraph And MongoDB](https://www.mongodb.com/company/blog/product-release-announcements/powering-long-term-memory-for-agents-langgraph)
- 출처: [Integrate MongoDB with LangGraph](https://www.mongodb.com/docs/atlas/ai-integrations/langgraph/)

### 3.5 Redis 통합

`langgraph-checkpoint-redis` 패키지는 Redis 기반 체크포인터와 스토어를 모두 제공한다:

**체크포인터**:
- `RedisSaver`: 전체 체크포인트 히스토리 유지
- `ShallowRedisSaver`: 메모리 최적화, 스레드당 최신 체크포인트만 보존

**스토어**:
- `RedisStore`: 벡터 검색 및 메타데이터 필터링 지원
- Redis 8.0 이상에서는 RedisJSON, RediSearch가 기본 내장

v0.1.0에서 저장 형식이 근본적으로 재설계되어 인라인 채널 값 방식을 도입했다(이전 버전 체크포인트와 비호환).

- 출처: [LangGraph Redis Checkpoint 0.1.0](https://redis.io/blog/langgraph-redis-checkpoint-010/)
- 출처: [LangGraph & Redis: Build smarter AI agents](https://redis.io/blog/langgraph-redis-build-smarter-ai-agents-with-memory-persistence/)
- 출처: [langgraph-checkpoint-redis GitHub](https://github.com/redis-developer/langgraph-redis)

### 3.6 활용 사례

- 고객 지원 에이전트: 채널 간 고객 선호도 기억
- 개인 비서 애플리케이션: 사용자 습관 학습
- 기업 지식 관리: 조직 지식 축적
- 멀티에이전트 시스템: 팀 간 학습 경험 공유

---

## 4. 기존 리서치 내용 (보존)

### 4.1 LangGraph 핵심 개념

#### StateGraph 클래스
- LangGraph의 주요 그래프 클래스
- TypedDict 또는 dataclass로 State 정의
- 노드와 엣지로 워크플로우 구성

#### 노드(Node)
- Python 함수로 정의
- state를 입력받아 업데이트된 state 반환
- `builder.add_node("name", function)` 으로 등록

#### 엣지(Edge)
- 정적 엣지: `builder.add_edge("node_a", "node_b")`
- 조건부 엣지: `builder.add_conditional_edges("node_a", routing_fn)`

### 4.2 조건부 분기 패턴

```python
def should_continue(state: State) -> str:
    if state["revision_count"] >= 3:
        return "end"
    return "revise" if state["feedback"] else "end"

builder.add_conditional_edges(
    "validate",
    should_continue,
    {"end": END, "revise": "revise_node"}
)
```

### 4.3 순환(Loop) 패턴

- LangGraph는 순환 그래프 지원 (DAG가 아님)
- 무한 루프 방지: 최대 반복 횟수, 조기 종료 조건
- `END` 노드로 그래프 종료

### 4.4 에러 처리 및 재시도

**재시도 정책**: 노드/태스크/그래프 수준에서 설정 가능. 지수 백오프 지원.

**폴백 패턴**:
- 노드 수준: try-except로 에러 객체를 상태에 기록
- 그래프 수준: 조건부 엣지로 error_handler 노드로 분기
- 앱 수준: 서킷 브레이커, 레이트 리미팅

**복구 전략**:
- 재시도 + 더 간단한 폴백 (가벼운 모델, 캐시 응답)
- Human-in-the-loop 에스컬레이션
- 상태 롤백 및 동적 재계획

### 4.5 컴파일 및 실행

```python
graph = builder.compile()
result = graph.invoke(initial_state)
```

---

## 참고 자료 (전체)

### 공식 문서 및 블로그
- LangGraph 1.0 GA 공지: https://changelog.langchain.com/announcements/langgraph-1-0-is-now-generally-available
- LangChain/LangGraph v1.0 블로그: https://blog.langchain.com/langchain-langgraph-1dot0/
- What's new in LangGraph v1: https://docs.langchain.com/oss/python/releases/langgraph-v1
- Graph API overview: https://docs.langchain.com/oss/python/langgraph/graph-api
- Memory overview: https://docs.langchain.com/oss/python/langgraph/memory
- Storage Reference: https://reference.langchain.com/python/langgraph/store/
- Trace LangGraph applications: https://docs.langchain.com/langsmith/trace-with-langgraph
- LangSmith Evaluation: https://docs.langchain.com/langsmith/evaluation
- LangSmith Pricing: https://www.langchain.com/pricing
- LangSmith Pricing FAQ: https://docs.langchain.com/langsmith/pricing-faq

### 메모리/스토어 통합
- Semantic Search for LangGraph Memory: https://blog.langchain.com/semantic-search-for-langgraph-memory/
- Launching Long-Term Memory Support: https://www.blog.langchain.com/launching-long-term-memory-support-in-langgraph/
- MongoDB + LangGraph: https://www.mongodb.com/company/blog/product-release-announcements/powering-long-term-memory-for-agents-langgraph
- MongoDB Atlas LangGraph Docs: https://www.mongodb.com/docs/atlas/ai-integrations/langgraph/
- Redis + LangGraph: https://redis.io/blog/langgraph-redis-build-smarter-ai-agents-with-memory-persistence/
- Redis Checkpoint 0.1.0: https://redis.io/blog/langgraph-redis-checkpoint-010/
- langgraph-checkpoint-redis GitHub: https://github.com/redis-developer/langgraph-redis
- Semantic Search How-To: https://langchain-ai.github.io/langgraph/how-tos/memory/semantic-search/

### 통계
- PyPI Download Stats: https://pypistats.org/packages/langgraph
- langgraph PyPI: https://pypi.org/project/langgraph/

### 트레이싱/디버깅
- LangSmith Observability: https://www.langchain.com/langsmith/observability
- Debugging Deep Agents: https://blog.langchain.com/debugging-deep-agents-with-langsmith/
- Advanced Tracing Techniques: https://sparkco.ai/blog/advanced-langsmith-agent-tracing-techniques-in-2025
- Connect traces to server logs: https://changelog.langchain.com/announcements/connect-traces-in-langsmith-to-server-logs-in-langgraph-platform

### 튜토리얼/가이드
- LangChain Graph API: https://docs.langchain.com/oss/python/langgraph/graph-api
- Real Python LangGraph Tutorial: https://realpython.com/langgraph-python/
- DataCamp LangGraph Agents: https://www.datacamp.com/tutorial/langgraph-agents
- Error Handling Best Practices: https://sparkco.ai/blog/advanced-error-handling-strategies-in-langgraph-applications
- Checkpointers vs Storage Discussion: https://github.com/langchain-ai/langgraph/discussions/3496
- LangGraph GitHub: https://github.com/langchain-ai/langgraph
