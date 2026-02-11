# 제6장: LangGraph 1.0으로 상태·분기·반복을 제어한다

## 학습 목표

1. LangGraph의 상태(State) 모델을 정의하고 노드 간 데이터를 전달한다.
2. LangGraph 1.0의 안정성 보장과 핵심 변경 사항을 설명한다.
3. 조건부 엣지(conditional edge)를 사용하여 워크플로우 분기를 구현한다.
4. LangSmith를 활용하여 LangGraph 워크플로우를 추적하고 디버깅한다.
5. 체크포인터와 Store API를 구분하여 단기·장기 메모리 전략을 설계한다.

## 선수 지식

- 5장에서 구현한 LangChain 에이전트 개념
- Python 기본 문법 및 TypedDict
- OpenAI API 사용 경험

---

## 6.1 LangGraph 소개: 그래프 기반 워크플로우의 필요성

5장에서 구현한 ReAct 에이전트는 단순한 도구 호출에 적합하지만, 복잡한 비즈니스 로직에는 한계가 있다. 조건 분기, 반복, 병렬 실행이 필요한 경우 LLM의 판단에만 의존하면 예측 불가능한 동작이 발생할 수 있다.

LangGraph는 노드(node)와 엣지(edge)로 워크플로우를 명시적으로 정의한다. 개발자가 제어 흐름을 직접 설계하므로 디버깅이 쉽고 동작을 예측할 수 있다.

**표 6.1** ReAct 에이전트와 LangGraph 비교

| 특성 | ReAct 에이전트 | LangGraph |
|-----|--------------|-----------|
| 제어 흐름 | LLM이 결정 | 개발자가 명시적 정의 |
| 조건 분기 | 프롬프트에 의존 | 조건부 엣지로 정확히 제어 |
| 상태 관리 | 메시지 히스토리 | 타입 안전한 State 객체 |
| 디버깅 | 로그 분석 필요 | 그래프 시각화 가능 |

LangGraph는 LangChain 위에 구축되어 있으며, 순환 그래프(cyclic graph)를 지원한다. 에이전트가 반복적으로 작업을 수행하고, 결과를 검증하고, 필요시 재시도하는 패턴을 자연스럽게 표현할 수 있다.

---

## 6.2 상태(State) 모델링

LangGraph의 핵심은 State 객체다. TypedDict를 사용하여 워크플로우 전체에서 공유되는 상태를 정의하고, 각 노드가 상태를 읽고 업데이트한다.

```python
class WorkflowState(TypedDict):
    draft: str
    feedback: Annotated[list[str], add]
    revision_count: int
```

_전체 코드는 practice/chapter6/code/6-6-langgraph-workflow.py 참고_

상태 정의에서 중요한 개념은 리듀서(reducer)다. 기본적으로 노드가 반환하는 값은 기존 상태를 덮어쓴다. 그러나 `Annotated`와 `operator.add`를 사용하면 리스트에 값을 누적할 수 있다. 이는 피드백 히스토리를 유지하는 데 유용하다.

상태는 워크플로우의 "메모리" 역할을 한다. 노드가 실행될 때마다 상태가 업데이트되고, 다음 노드로 전달된다. 타입 힌트를 통해 각 필드가 올바른 타입인지 검증할 수 있다.

---

## 6.3 조건 분기: 검증 결과에 따른 경로 선택

조건부 엣지(conditional edge)를 사용하면 상태에 따라 다른 노드로 분기할 수 있다. 분기 조건을 결정하는 함수를 정의하고, `add_conditional_edges`로 등록한다.

```python
def should_continue(state: WorkflowState) -> str:
    if state["revision_count"] >= 3:
        return "end"
    return "revise" if state["feedback"] else "end"
```

_전체 코드는 practice/chapter6/code/6-6-langgraph-workflow.py 참고_

분기 함수는 현재 상태를 분석하고 다음 노드의 이름을 문자열로 반환한다. 반환값과 실제 노드를 매핑하는 딕셔너리를 함께 전달한다.

```python
builder.add_conditional_edges(
    "validate",
    should_continue,
    {"end": END, "revise": "revise"}
)
```

이 패턴을 사용하면 검증 통과 시 종료하고, 실패 시 수정 노드로 이동하는 분기를 명확하게 표현할 수 있다.

---

## 6.4 반복 루프: 재시도와 검증 루프

LangGraph는 순환 그래프를 지원하므로 "생성 → 검증 → 실패 시 수정" 사이클을 반복할 수 있다. 무한 루프를 방지하기 위해 최대 반복 횟수를 설정하는 것이 중요하다.

실습에서 구현한 워크플로우는 다음과 같은 흐름을 따른다.

1. `generate_draft`: 초안 생성
2. `validate`: 초안 검증, 피드백 생성
3. 조건 분기: 통과 시 종료, 실패 시 `revise`로 이동
4. `revise`: 피드백 반영하여 수정
5. 다시 `validate`로 이동 (최대 3회 반복)

이 패턴의 핵심은 `revise` 노드가 `validate` 노드로 다시 연결된다는 점이다. 이렇게 순환 구조를 만들되, 조건 분기에서 종료 조건을 명확히 정의하여 무한 루프를 방지한다.

---

## 6.5 실패 복구: 에러 핸들링과 폴백

노드 실행 중 에러가 발생할 수 있다. API 호출 실패, 타임아웃, 잘못된 응답 등 다양한 상황에 대비해야 한다. 에러 처리는 노드 내부에서 try-except로 구현한다.

에러가 발생하면 상태에 에러 정보를 기록하고, 조건 분기에서 에러 상태를 확인하여 적절한 경로로 이동한다. 폴백 전략으로는 더 간단한 모델 사용, 캐시된 응답 반환, 또는 사람에게 에스컬레이션하는 방법이 있다.

LangGraph는 노드 수준에서 재시도 정책을 설정할 수도 있다. 지수 백오프(exponential backoff)를 적용하여 일시적인 오류에서 자동으로 복구할 수 있다.

---

## 6.6 LangGraph 1.0: 첫 안정 릴리스

LangGraph 1.0은 2025년 10월 22일 일반 공개(GA)되었다. LangChain 1.0과 동시에 발표되었으며, Uber, LinkedIn, Klarna 등 기업에서 1년 이상 프로덕션 운영한 결과를 바탕으로 안정화된 릴리스다.

### 6.6.1 API 안정성 보장

v1.0의 핵심 가치는 새 기능 추가가 아니라 **안정성 확정**이다. v0.6.6에서 v1.0으로의 업그레이드에서 브레이킹 체인지가 전혀 없으며, **v2.0까지 하위 호환성**을 공식 보장한다. 유일한 변경점은 `langgraph.prebuilt.create_react_agent`가 `langchain.agents.create_agent`로 이전된 것이며, 기존 함수의 제거는 v2.0에서 예정이다.

**표 6.2** LangGraph pre-1.0 vs 1.0 주요 변경

| 항목 | pre-1.0 (0.x) | v1.0 |
|------|---------------|------|
| API 안정성 | 실험적, 수시 변경 | v2.0까지 안정 보장 |
| 내구 실행(Durable Execution) | 존재했으나 비공식 | 공식 지원, 프로덕션 검증 |
| 스트리밍 | 기본 지원 | LLM 토큰·도구 호출·상태 전이 전체 스트리밍 |
| Human-in-the-loop | 패턴으로 존재 | 1등급 API(interrupt → resume) |
| Functional API | 미존재 | `@task`, `@entrypoint` 데코레이터 추가 |
| 메모리 | 체크포인터 중심 | 단기(체크포인터) + 장기(Store API) 이원 체계 |

### 6.6.2 Functional API

v1.0에서 추가된 Functional API는 `@task`와 `@entrypoint` 데코레이터로 워크플로우를 정의한다. 기존 StateGraph 방식과 달리 Python 함수를 직접 조합하므로, 단순한 파이프라인에서 그래프 정의 없이도 내구 실행(durable execution)의 이점을 누릴 수 있다. StateGraph가 명시적 제어 흐름에 적합하다면, Functional API는 익숙한 함수 호출 패턴으로 빠르게 프로토타이핑할 때 유리하다.

---

## 6.7 LangSmith 통합: 워크플로우 추적과 평가

LangGraph 워크플로우가 복잡해질수록 "어떤 노드에서 무엇이 일어났는가"를 추적하는 것이 중요해진다. LangSmith는 LangGraph와 네이티브 통합되어, 환경 변수 설정만으로 트레이싱이 활성화된다.

```python
import os
os.environ["LANGSMITH_TRACING"] = "true"
os.environ["LANGSMITH_API_KEY"] = "ls__..."
os.environ["LANGSMITH_PROJECT"] = "my-langgraph-project"
```

### 6.7.1 중첩 스팬(Nested Span) 트레이싱

LangSmith의 트레이싱 모델은 트리 구조를 따른다. 하나의 **트레이스**(Trace)는 에이전트의 단일 실행 단위이며, 그 안에 여러 **런**(Run)이 부모-자식 관계로 중첩된다. LLM 호출, 도구 호출, 노드 전환 등 각 단계의 입력·출력·소요 시간을 확장하여 확인할 수 있다. 10개 이상의 도구 호출이 포함된 멀티스텝 에이전트도 즉시 재생(replay)하여 병목 지점을 식별할 수 있다.

커스텀 함수에 `@traceable` 데코레이터를 적용하면, 모델 호출 옆에 커스텀 로직도 트레이스에 포함된다. 6.10절의 디버깅에서 "각 노드에서 상태를 로깅한다"고 했는데, LangSmith를 사용하면 별도 로깅 코드 없이도 이 정보를 자동으로 수집할 수 있다.

### 6.7.2 평가 워크플로우

LangSmith는 오프라인/온라인 양면의 평가 체계를 제공한다. 오프라인 평가는 데이터셋을 구성하고(수동 큐레이션, 프로덕션 트레이스, 합성 데이터), 에이전트를 전체 데이터셋에 대해 실행한 뒤, 평가자(evaluator)로 점수를 매기는 방식이다. 평가자는 LLM-as-judge, 코드 기반 휴리스틱, 커스텀 로직을 혼합하여 사용할 수 있다. 버전 고정 데이터셋으로 프롬프트나 모델 변경 시 성능 회귀를 감지한다.

---

## 6.8 LangGraph + 장기 메모리

6.2절에서 다룬 상태(State)는 하나의 워크플로우 실행 내에서만 유효하다. 그러나 실제 에이전트 애플리케이션은 대화 간에도 정보를 유지해야 한다. LangGraph는 이를 위해 **체크포인터**(Checkpointer)와 **스토어**(Store)라는 두 가지 메모리 계층을 제공한다.

### 6.8.1 체크포인터 vs 스토어

**표 6.3** 메모리 계층 비교

| 구분 | 체크포인터(Checkpointer) | 스토어(Store) |
|------|--------------------------|---------------|
| 범위 | 단일 스레드(대화) 내 | 스레드 간(cross-thread) |
| 용도 | 멀티턴 대화 컨텍스트, 중단 복구 | 사용자 선호, 학습된 지식, 과거 결정 |
| 데이터 | 그래프 실행 상태의 스냅샷 | JSON 문서(key-value) |
| 수명 | 스레드 종료 시 의미 감소 | 애플리케이션 수명과 동일 |
| 검색 | thread_id + checkpoint_id | namespace + key, 또는 의미 검색 |

체크포인터는 "이 대화에서 어디까지 진행했는가"를 다루고, 스토어는 "이 사용자에 대해 무엇을 알고 있는가"를 다룬다. 체크포인터만으로는 대화를 넘어선 학습이 불가능하므로, 장기 메모리가 필요한 에이전트는 두 계층을 함께 사용해야 한다.

### 6.8.2 Store API

LangGraph의 `BaseStore` 인터페이스는 세 가지 핵심 메서드를 제공한다.

```python
store.put(("users", "user_123"), "pref", {"language": "ko"})
item = store.get(("users", "user_123"), "pref")
results = store.search(("users", "user_123"), query="선호 언어")
```

`put`으로 네임스페이스에 JSON 문서를 저장하고, `get`으로 키 기반 조회, `search`로 필터 또는 의미 검색(semantic search)을 수행한다. 네임스페이스는 `("users", "user_123", "preferences")`처럼 계층적으로 구성할 수 있다.

### 6.8.3 외부 저장소 통합

프로덕션 환경에서는 `InMemoryStore` 대신 영속적인 외부 저장소를 사용한다. 대표적인 선택지는 다음과 같다.

**MongoDB 통합**: `langgraph-store-mongodb` 패키지가 `MongoDBStore`를 제공한다. MongoDB의 네이티브 JSON 문서 구조와 LangGraph의 메모리 형식이 직접 매핑되며, Atlas Vector Search로 의미 기반 메모리 검색을 지원한다. TTL 인덱스로 오래된 메모리를 자동 정리할 수도 있다.

**Redis 통합**: `langgraph-checkpoint-redis` 패키지는 체크포인터(`RedisSaver`, `ShallowRedisSaver`)와 스토어(`RedisStore`)를 모두 제공한다. Redis 8.0 이상에서는 RedisJSON과 RediSearch가 기본 내장되어, 벡터 검색과 메타데이터 필터링을 별도 설정 없이 사용할 수 있다.

---

## 6.9 실습: 초안 생성 → 검증 → 수정 워크플로우

### 실습 목표

1. LangGraph로 순환 워크플로우를 구현한다.
2. OpenAI API를 사용하여 초안을 생성하고, 검증하고, 수정한다.
3. 최대 3회 수정 후 최종 결과를 파일로 저장한다.

### 실습 환경 설정

```bash
cd practice/chapter6
python3 -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r code/requirements.txt
cp code/.env.example code/.env  # OPENAI_API_KEY 설정
python3 code/6-6-langgraph-workflow.py
```

### 워크플로우 구성

```
[START] → [generate_draft] → [validate] → {조건 분기}
                                              ↓ (통과)
                                           [END]
                                              ↓ (실패 & 수정 횟수 < 3)
                                           [revise] → [validate] → ...
```

### 실행 결과

실제 실행 시 생성된 초안:

```
인공지능(AI)은 소프트웨어 개발의 패러다임을 변화시키고 있습니다.
최근 연구에 따르면, AI 도구를 활용한 개발 팀은 전통적인 방법에 비해
코드 작성 속도가 최대 40% 향상될 수 있습니다...
```

**표 6.4** 워크플로우 실행 결과

| 항목 | 값 |
|-----|-----|
| 주제 | 인공지능이 소프트웨어 개발에 미치는 영향 |
| 수정 횟수 | 0 |
| 검증 통과 | true |
| 실행 시간 | 약 8초 |

### 실습 산출물

- `practice/chapter6/data/output/ch06_workflow_log.txt`: 워크플로우 실행 로그
- `practice/chapter6/data/output/ch06_draft_v1.txt`: 초기 초안
- `practice/chapter6/data/output/ch06_draft_final.txt`: 최종 결과물
- `practice/chapter6/data/output/ch06_result.json`: 실행 결과 요약

---

## 6.10 실패 사례와 디버깅

### 무한 루프 발생

검증 함수가 항상 실패를 반환하거나, 종료 조건이 누락되면 무한 루프에 빠질 수 있다. 이를 방지하려면 반드시 최대 반복 횟수를 설정하고, 조건 분기에서 이를 확인해야 한다.

```python
if state["revision_count"] >= MAX_REVISIONS:
    return "end"  # 강제 종료
```

### 상태 누락 오류

노드가 필수 상태 필드를 업데이트하지 않으면 다음 노드에서 오류가 발생한다. TypedDict를 사용하면 타입 검사기가 이러한 문제를 미리 발견할 수 있다.

### 디버깅 팁

1. 각 노드에서 상태를 로깅한다
2. 조건 분기 함수에서 결정 이유를 기록한다
3. 그래프를 시각화하여 흐름을 확인한다

---

## 핵심 정리

- LangGraph는 노드와 엣지로 워크플로우를 명시적으로 정의한다.
- LangGraph 1.0(2025.10)은 v2.0까지 API 안정성을 보장하는 첫 프로덕션 릴리스다.
- TypedDict로 상태를 정의하고, Annotated로 리듀서 동작을 지정한다.
- 조건부 엣지로 상태에 따른 분기를 구현한다.
- 순환 그래프로 재시도/검증 루프를 만들되, 최대 반복 횟수로 무한 루프를 방지한다.
- LangSmith 연동으로 중첩 스팬 트레이싱, 평가 워크플로우를 수행한다.
- 체크포인터(단기)와 Store API(장기)의 이원 메모리 체계로 대화를 넘어선 학습이 가능하다.

---

## 다음 장 예고

7장에서는 멀티에이전트 시스템을 다룬다. 여러 에이전트가 협력하여 복잡한 작업을 수행하는 패턴과, 에이전트 프레임워크(CrewAI, AutoGen, OpenAI Agents SDK) 비교, A2A 프로토콜 실전 적용, 로우코드 에이전트 빌더를 살펴본다.

---

## 참고문헌

LangChain. (2025). *LangGraph 1.0 is now generally available*. https://changelog.langchain.com/announcements/langgraph-1-0-is-now-generally-available

LangChain. (2025). *LangChain and LangGraph Agent Frameworks Reach v1.0 Milestones*. https://blog.langchain.com/langchain-langgraph-1dot0/

LangChain. (2025). *What's new in LangGraph v1*. https://docs.langchain.com/oss/python/releases/langgraph-v1

LangChain. (2025). *Graph API - LangGraph Documentation*. https://docs.langchain.com/oss/python/langgraph/graph-api

LangChain. (2025). *Memory overview - LangGraph Documentation*. https://docs.langchain.com/oss/python/langgraph/memory

LangChain. (2025). *Trace LangGraph applications with LangSmith*. https://docs.langchain.com/langsmith/trace-with-langgraph

LangChain. (2025). *LangSmith Evaluation*. https://docs.langchain.com/langsmith/evaluation

LangChain. (2025). *Semantic Search for LangGraph Memory*. https://blog.langchain.com/semantic-search-for-langgraph-memory/

MongoDB. (2025). *Powering Long-Term Memory For Agents With LangGraph And MongoDB*. https://www.mongodb.com/company/blog/product-release-announcements/powering-long-term-memory-for-agents-langgraph

Redis. (2025). *LangGraph & Redis: Build smarter AI agents with memory persistence*. https://redis.io/blog/langgraph-redis-build-smarter-ai-agents-with-memory-persistence/
