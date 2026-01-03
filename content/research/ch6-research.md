# 6장 리서치 결과: LangGraph로 상태·분기·반복을 제어한다

## 조사일: 2026-01-02

## 1. LangGraph 핵심 개념

### StateGraph 클래스
- LangGraph의 주요 그래프 클래스
- TypedDict 또는 dataclass로 State 정의
- 노드와 엣지로 워크플로우 구성

### 상태(State) 정의
```python
from typing import TypedDict, Annotated
from operator import add

class WorkflowState(TypedDict):
    messages: list
    draft: str
    feedback: Annotated[list[str], add]  # 리스트 누적
    revision_count: int
```

### 노드(Node)
- Python 함수로 정의
- state를 입력받아 업데이트된 state 반환
- `builder.add_node("name", function)` 으로 등록

### 엣지(Edge)
- 정적 엣지: `builder.add_edge("node_a", "node_b")`
- 조건부 엣지: `builder.add_conditional_edges("node_a", routing_fn)`

## 2. 조건부 분기 패턴

### add_conditional_edges
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

## 3. 순환(Loop) 패턴

### 재시도/검증 루프
- LangGraph는 순환 그래프 지원
- 무한 루프 방지: 최대 반복 횟수, 조기 종료 조건
- `END` 노드로 그래프 종료

## 4. 에러 처리 및 재시도

### 재시도 정책
- 노드/태스크/그래프 수준에서 설정 가능
- 지수 백오프(exponential backoff) 지원
- 재시도 소진 시 에러 상태 기록

### 폴백 패턴
- 노드 수준: try-except로 에러 객체를 상태에 기록
- 그래프 수준: 조건부 엣지로 error_handler 노드로 분기
- 앱 수준: 서킷 브레이커, 레이트 리미팅

### 복구 전략
- 재시도 + 더 간단한 폴백 (가벼운 모델, 캐시 응답)
- Human-in-the-loop 에스컬레이션
- 상태 롤백 및 동적 재계획

## 5. 컴파일 및 실행

### 그래프 컴파일
```python
graph = builder.compile()
result = graph.invoke(initial_state)
```

### 체크포인터
- 상태 지속성을 위한 체크포인터 설정
- 재시도 및 복구에 활용

## 참고 자료

- LangChain Graph API: https://docs.langchain.com/oss/python/langgraph/graph-api
- Real Python LangGraph Tutorial: https://realpython.com/langgraph-python/
- DataCamp LangGraph Agents: https://www.datacamp.com/tutorial/langgraph-agents
- LangChain Blog: https://blog.langchain.com/langgraph/
- Error Handling Best Practices: https://sparkco.ai/blog/advanced-error-handling-strategies-in-langgraph-applications
