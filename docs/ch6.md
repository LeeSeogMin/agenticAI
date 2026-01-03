# 제6장: LangGraph로 상태·분기·반복을 제어한다

## 학습 목표

1. LangGraph의 상태(State) 모델을 정의하고 노드 간 데이터를 전달한다.
2. 조건부 엣지(conditional edge)를 사용하여 워크플로우 분기를 구현한다.
3. 반복 루프를 구성하여 검증-재시도 패턴을 적용한다.
4. 실패 복구 전략을 설계하고 에러 핸들링을 구현한다.
5. 실습 산출물(워크플로우 실행 로그, 생성된 문서)을 저장하고 해석한다.

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

## 6.6 실습: 초안 생성 → 검증 → 수정 워크플로우

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

**표 6.2** 워크플로우 실행 결과

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

## 6.7 실패 사례와 디버깅

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
- TypedDict로 상태를 정의하고, Annotated로 리듀서 동작을 지정한다.
- 조건부 엣지로 상태에 따른 분기를 구현한다.
- 순환 그래프로 재시도/검증 루프를 만들되, 최대 반복 횟수로 무한 루프를 방지한다.
- 에러는 상태에 기록하고, 조건 분기에서 적절히 처리한다.

---

## 다음 장 예고

7장에서는 멀티에이전트 시스템을 다룬다. 여러 에이전트가 협력하여 복잡한 작업을 수행하는 패턴과, CrewAI, AutoGen 등 프레임워크 선택 기준을 살펴본다.

---

## 참고문헌

LangChain. (2025). *Graph API - LangGraph Documentation*. https://docs.langchain.com/oss/python/langgraph/graph-api

Real Python. (2025). *LangGraph: Build Stateful AI Agents in Python*. https://realpython.com/langgraph-python/

DataCamp. (2025). *How to Build LangGraph Agents Hands-On Tutorial*. https://www.datacamp.com/tutorial/langgraph-agents

LangChain Blog. (2024). *LangGraph*. https://blog.langchain.com/langgraph/
