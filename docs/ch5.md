# 제5장: LangChain과 MCP의 연결 — 도구를 쓰는 에이전트

## 학습 목표

1. LangChain 에이전트가 도구를 선택하고 호출하는 흐름을 설명한다.
2. 툴 선택 실패를 줄이기 위한 프롬프트 구조와 도구 설명 작성 원칙을 적용한다.
3. 에이전트 실행 과정을 로그와 트레이스로 추적 가능하게 만든다.
4. 도구 입력을 검증하여 잘못된 호출을 사전에 차단한다.
5. 실습 산출물(에이전트 실행 로그, 최종 결과 JSON)을 저장하고 해석한다.

## 선수 지식

- 4장에서 구현한 MCP 서버 개념
- Python 기본 문법 및 비동기 프로그래밍
- OpenAI API 사용 경험

---

## 5.1 에이전트와 도구의 관계: LLM이 도구를 선택하는 방식

에이전트(agent)가 도구를 "알아서" 선택하는 것처럼 보이지만, 실제로는 대규모 언어 모델(Large Language Model, LLM)이 도구 설명을 읽고 적합한 도구를 판단하는 과정이다. 에이전트는 사용자의 요청을 분석하고, 주어진 도구 목록에서 적절한 도구를 선택하여 호출한 뒤, 그 결과를 바탕으로 다음 행동을 결정한다.

이 과정은 ReAct(Reasoning and Acting) 패턴으로 알려져 있다. 에이전트는 "생각(Thought) → 행동(Action) → 관찰(Observation)" 루프를 반복하며 목표를 달성한다.

**표 5.1** 에이전트 실행 루프의 단계

| 단계 | 설명 | 예시 |
|-----|------|------|
| 생각(Thought) | 현재 상황을 분석하고 다음 행동 결정 | "날씨 정보가 필요하다" |
| 행동(Action) | 선택한 도구와 입력 파라미터 지정 | `get_weather(lat=37.5, lon=127)` |
| 관찰(Observation) | 도구 실행 결과 확인 | `{"temp": 15, "condition": "Clear"}` |
| 반복/종료 | 목표 달성 여부 판단 | 최종 답변 생성 또는 다음 행동 |

도구 설명이 LLM의 도구 선택에 결정적인 영향을 미친다. 설명이 모호하거나 부정확하면 잘못된 도구를 선택하거나, 불필요한 도구를 호출하는 문제가 발생한다. 도구가 여러 개일 때 유사한 기능을 가진 도구들 사이에서 선택 충돌이 발생할 수도 있다.

---

## 5.2 툴 선택 실패를 줄이는 프롬프트 구조

도구 선택 실패는 크게 세 가지 유형으로 분류된다.

1. **잘못된 도구 선택**: 의도와 다른 도구를 호출
2. **불필요한 도구 호출**: 도구 없이 답변 가능한데 도구를 호출
3. **필요한 도구 미호출**: 도구가 필요한데 직접 답변 시도

이러한 실패를 줄이기 위해 도구 설명을 명확하게 작성해야 한다. 도구 설명에는 목적, 입력, 출력, 제약 조건을 포함한다.

```python
@tool
def get_weather_tool(latitude: float, longitude: float) -> str:
    """지정된 위도/경도의 날씨 조회. 도시 이름만 있으면 좌표 먼저 조회."""
    return get_weather(latitude, longitude)
```

_전체 코드는 practice/chapter5/code/5-5-langchain-agent.py 참고_

유사한 도구가 있을 때는 각 도구의 차이점을 명확히 기술한다. 예를 들어 도시 좌표 조회 도구와 날씨 조회 도구가 있다면, "도시 이름만 있을 때는 먼저 좌표를 조회하세요"와 같이 호출 순서를 안내한다.

시스템 프롬프트에서 도구 사용 지침을 명시하는 것도 효과적이다. 에이전트가 어떤 순서로 도구를 사용해야 하는지, 어떤 상황에서 도구를 사용하지 말아야 하는지를 구체적으로 설명한다.

---

## 5.3 관측 가능성: 로그와 트레이스

에이전트가 "블랙박스"처럼 동작하면 디버깅이 어렵다. LangChain의 콜백(callback) 시스템을 활용하면 에이전트의 각 단계를 추적할 수 있다. 콜백 핸들러는 체인 시작/종료, 도구 호출, 에러 발생 등의 이벤트를 캡처한다.

```python
class AgentLoggingCallback(BaseCallbackHandler):
    def on_tool_start(self, serialized, input_str, **kwargs):
        logger.info(f"도구 호출: {serialized.get('name')}")
```

_전체 코드는 practice/chapter5/code/5-5-langchain-agent.py 참고_

운영 환경에서는 LangSmith나 Langfuse 같은 관측 가능성 플랫폼을 활용할 수 있다. 이러한 도구는 토큰 사용량, 응답 시간, 실패 추적 등을 자동으로 기록한다. LangSmith는 환경 변수 하나만 설정하면 트레이싱이 시작되며, 애플리케이션 레이턴시에 영향을 주지 않는다.

로깅 전략에서 중요한 것은 실패 시 컨텍스트를 보존하는 것이다. 어떤 입력이 주어졌을 때 어떤 도구가 호출되었고, 그 결과가 무엇이었는지를 기록해야 문제 재현과 개선이 가능하다.

---

## 5.4 입력 검증: 잘못된 도구 호출 방지

LLM이 생성한 도구 입력이 항상 올바른 것은 아니다. 타입 오류, 범위 초과, 필수 파라미터 누락 등의 문제가 발생할 수 있다. Pydantic을 활용하면 도구 입력 스키마를 정의하고 자동으로 검증할 수 있다.

```python
from pydantic import BaseModel, Field

class WeatherInput(BaseModel):
    latitude: float = Field(..., ge=-90, le=90)
    longitude: float = Field(..., ge=-180, le=180)
```

_전체 코드는 practice/chapter5/code/5-5-langchain-agent.py 참고_

검증 실패 시 명확한 에러 메시지를 반환하면 에이전트가 이를 보고 재시도할 수 있다. 예를 들어 위도가 범위를 벗어났다면 "위도는 -90에서 90 사이여야 합니다"라는 메시지를 반환한다. LLM은 이 피드백을 바탕으로 올바른 값으로 다시 시도한다.

---

## 5.5 실습: OpenAI + LangChain으로 도구를 쓰는 에이전트 만들기

### 실습 목표

1. OpenAI API와 LangChain을 사용하여 에이전트를 구현한다.
2. 3개의 도구(도시 좌표 조회, 날씨 조회, 파일 저장)를 연결한다.
3. 에이전트 실행 과정을 로그로 기록하고, 최종 결과를 JSON 파일로 저장한다.

### 도구 구성

1. **도시 좌표 조회 도구(get_city_coordinates_tool)**: 도시 이름을 받아 위도/경도 반환
2. **날씨 조회 도구(get_weather_tool)**: 위도/경도를 받아 현재 날씨 정보 반환 (Open-Meteo API 사용)
3. **파일 저장 도구(save_to_file_tool)**: 텍스트 내용을 지정된 파일에 저장

### 실습 환경 설정

```bash
cd practice/chapter5
python3 -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r code/requirements.txt
cp code/.env.example code/.env  # OPENAI_API_KEY 설정
python3 code/5-5-langchain-agent.py
```

### 에이전트 실행 흐름

실습 코드를 실행하면 에이전트는 다음 순서로 도구를 호출한다.

1. 사용자 입력: "서울의 현재 날씨를 조회하고, 결과를 ch05_weather_report.txt 파일로 저장해주세요."
2. 에이전트가 `get_city_coordinates_tool`을 호출하여 서울의 좌표 획득
3. 획득한 좌표로 `get_weather_tool`을 호출하여 날씨 데이터 조회
4. 날씨 정보를 정리하여 `save_to_file_tool`로 파일 저장
5. 최종 응답 생성

### 실행 결과

**표 5.2** 에이전트 도구 호출 순서

| 순서 | 도구 | 입력 | 출력 |
|-----|------|------|------|
| 1 | get_city_coordinates_tool | `{"city_name": "Seoul"}` | `{"latitude": 37.5665, "longitude": 126.978}` |
| 2 | get_weather_tool | `{"latitude": 37.5665, "longitude": 126.978}` | 기온, 습도, 날씨 상태 |
| 3 | save_to_file_tool | `{"filename": "ch05_weather_report.txt", ...}` | 저장 성공 |

실제 실행 시 생성된 날씨 보고서:

```
서울의 현재 날씨:
온도: -7.4°C
습도: 42%
날씨 상태: 맑음
풍속: 7.8 km/h
```

### 실습 산출물

- `practice/chapter5/data/output/ch05_agent_log.txt`: 에이전트 실행 로그
- `practice/chapter5/data/output/ch05_result.json`: 최종 실행 결과
- `practice/chapter5/data/output/ch05_weather_report.txt`: 에이전트가 생성한 날씨 보고서

---

## 5.6 실패 사례와 디버깅

### 잘못된 도구 선택

사용자가 "서울 날씨"라고 요청했을 때, 에이전트가 좌표 조회 없이 바로 `get_weather_tool`을 호출하면 실패한다. 이 문제는 도구 설명에 "도시 이름만 있다면 먼저 좌표를 조회하세요"라는 지침을 추가하여 해결한다.

로그를 분석하면 에이전트가 어떤 도구를 어떤 순서로 호출했는지 확인할 수 있다.

```
[tool_start] {"tool": "get_city_coordinates_tool", "input": "{'city_name': 'Seoul'}"}
[tool_start] {"tool": "get_weather_tool", "input": "{'latitude': 37.5665, 'longitude': 126.978}"}
```

### 무한 루프 방지

에이전트가 도구 호출과 재시도를 반복하며 무한 루프에 빠질 수 있다. 이를 방지하기 위해 최대 반복 횟수를 설정한다. LangGraph의 `create_react_agent`는 기본적으로 반복 횟수 제한을 제공한다.

### 토큰 한도 초과

대화가 길어지면 컨텍스트 윈도우 한도를 초과할 수 있다. 이 경우 이전 대화를 요약하거나, 중요한 정보만 유지하는 메모리 관리 전략이 필요하다.

---

## 핵심 정리

- 에이전트는 ReAct 패턴(Thought → Action → Observation)으로 도구를 선택하고 호출한다.
- 도구 설명에 목적, 입력, 출력, 제약을 명확히 기술하면 도구 선택 실패를 줄일 수 있다.
- 콜백 핸들러를 활용하면 에이전트 실행 과정을 추적하고 디버깅할 수 있다.
- Pydantic으로 도구 입력을 검증하면 잘못된 호출을 사전에 차단할 수 있다.
- 무한 루프 방지를 위해 최대 반복 횟수를 설정한다.

---

## 다음 장 예고

6장에서는 LangGraph를 사용하여 더 복잡한 에이전트 워크플로우를 구현한다. 조건부 분기, 병렬 실행, 상태 관리 등 고급 패턴을 다룬다.

---

## 참고문헌

LangChain. (2025). *Tools - LangChain Documentation*. https://docs.langchain.com/oss/python/langchain/tools

LangChain. (2025). *LangSmith Observability*. https://docs.langchain.com/oss/python/langgraph/observability

OpenAI. (2025). *How to build a tool-using agent with LangChain*. OpenAI Cookbook. https://cookbook.openai.com/examples/how_to_build_a_tool-using_agent_with_langchain

Langfuse. (2025). *Open Source Observability for LangChain*. https://langfuse.com/integrations/frameworks/langchain
