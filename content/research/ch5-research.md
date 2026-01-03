# 5장 리서치 결과: LangChain과 MCP의 연결

## 조사일: 2026-01-02

## 1. LangChain Agent Tool Calling

### 핵심 API
- `create_openai_tools_agent()`: OpenAI 도구 기반 에이전트 생성
- `ChatModel.bind_tools()`: 도구 정의를 모델에 연결
- `AIMessage.tool_calls`: 모델이 결정한 도구 호출 정보 접근

### LangChain 1.0 (2025년 11월)
- 표준화된 도구 호출 인터페이스
- 미들웨어로 에이전트 루프 커스터마이징
- 구조화된 출력 세밀 제어 (tool calling 또는 provider-native)

### 도구 정의 패턴
```python
@tool
def my_tool(param: str) -> str:
    """도구 설명 - 모델이 도구 선택 시 참조"""
    return result

tools = [my_tool]
agent = create_openai_tools_agent(llm, tools, prompt)
```

## 2. Observability & Callbacks

### LangSmith 통합
- 환경 변수 하나로 트레이싱 시작
- 비동기 처리로 애플리케이션 레이턴시 없음
- 실시간 모니터링, 알림, 사용량 분석

### 콜백 핸들러
```python
config={"callbacks": [langfuse_handler]}
```
- 트레이스, 토큰, 타이밍 캡처
- StdOutCallbackHandler로 콘솔 출력
- 커스텀 콜백으로 로깅 제어

### OpenTelemetry 통합
- 분산 추적, 메트릭, 로그 수집
- 기존 APM 도구와 연동 가능

## 3. Pydantic 입력 검증

### 구조화된 출력 방법
1. **Pydantic**: 가장 강력하고 엄격한 검증
2. **TypedDict**: 중간 수준 검증
3. **JSON Schema**: 단순한 스키마 정의

### 주요 API
```python
from pydantic import BaseModel, Field

class WeatherInput(BaseModel):
    latitude: float = Field(..., ge=-90, le=90)
    longitude: float = Field(..., ge=-180, le=180)
```

### 검증 실패 처리
- `parse()` 실패 시 ValidationError 발생
- 에이전트에 에러 피드백하여 재시도 유도

## 4. 에이전트 실행 패턴

### ReAct 패턴 (Thought → Action → Observation)
```
Question: 서울 날씨 알려줘
Thought: 날씨 정보가 필요하다. get_weather 도구 사용
Action: get_weather
Action Input: {"latitude": 37.5665, "longitude": 126.978}
Observation: {"temp": 15, "condition": "Clear"}
Thought: 정보를 얻었다. 답변 생성
Final Answer: 서울의 현재 기온은 15도이며 맑은 날씨입니다.
```

### AgentExecutor
```python
from langchain.agents import AgentExecutor

agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    verbose=True,
    max_iterations=10  # 무한 루프 방지
)
result = agent_executor.invoke({"input": query})
```

## 참고 자료

- LangChain Tools Documentation: https://docs.langchain.com/oss/python/langchain/tools
- OpenAI Cookbook - Tool-Using Agent: https://cookbook.openai.com/examples/how_to_build_a_tool-using_agent_with_langchain
- LangSmith Observability: https://docs.langchain.com/oss/python/langgraph/observability
- Langfuse Integration: https://langfuse.com/integrations/frameworks/langchain
- LangChain Structured Output: https://docs.langchain.com/oss/python/langchain/structured-output
