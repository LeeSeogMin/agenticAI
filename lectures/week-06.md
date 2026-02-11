# 6주차. MCP 서버 구현 기초

> **1회차** (강의 90분): MCP 서버 내부 구조(Tool/Resource/Prompt 상세), Python SDK 소개, 데코레이터 패턴, 도구 설명 작성법
> **2회차** (실습 90분): 날씨 API MCP 서버 구현 (환경 설정 - 도구 등록 - 리소스 등록 - 서버 실행 - MCP Inspector 테스트)

---

## 학습목표

1. MCP 서버의 세 가지 핵심 요소(Tool, Resource, Prompt)의 역할과 차이를 설명할 수 있다
2. Python MCP SDK의 FastMCP 패턴과 데코레이터 기반 등록 방식을 이해한다
3. LLM이 도구를 올바르게 선택하도록 이름, 설명, 파라미터 스키마를 설계할 수 있다
4. 날씨 API를 래핑하는 MCP 서버를 처음부터 끝까지 구현하고 Inspector로 검증한다

## 선수지식

- 5주차: MCP 서버 연결과 활용 (`.vscode/mcp.json` 설정, 기존 MCP 서버 사용 경험)
- 4주차: MCP 아키텍처 개념 (클라이언트-서버 구조, 전송 방식, 3대 프리미티브 개요)
- Python 기초 문법: 함수 정의, 타입 힌트, 독스트링, `async`/`await` 기본 개념
- VS Code 기본 사용법

---

## 1회차: 강의

### 6.1 왜 MCP 서버를 직접 만들어야 하는가

5주차에서는 이미 만들어진 MCP 서버를 Copilot에 연결하는 방법을 다루었다. GitHub MCP 서버, Playwright MCP 서버, Memory MCP 서버처럼 커뮤니티가 제공하는 도구를 설치하고 사용하는 것만으로도 Copilot의 능력을 크게 확장할 수 있었다. 그러나 기존 서버만으로는 자신의 업무에 특화된 기능을 제공하기 어렵다.

예를 들어, 학교의 학식 메뉴를 조회하거나, 사내 ERP 시스템에서 재고를 확인하거나, 특정 오픈 데이터 포털에서 실시간 정보를 가져오는 기능은 커뮤니티 서버에 존재하지 않는다. 이러한 기능이 필요할 때, MCP 서버를 직접 만들 수 있다는 것은 곧 Copilot에 원하는 능력을 자유롭게 추가할 수 있다는 뜻이다.

이번 주차에서는 MCP 서버의 내부를 열어 보고 직접 만들어 본다. 서버의 구조를 이해하고, Python SDK를 활용하여 첫 번째 MCP 서버를 구현하는 것이 목표이다.

### 6.2 MCP 서버의 세 가지 핵심 요소

MCP 서버는 세 가지 핵심 요소(Primitive)로 구성된다. 첫째는 도구(Tool), 둘째는 리소스(Resource), 셋째는 프롬프트(Prompt)이다. 이 세 요소의 역할을 정확히 이해하는 것이 MCP 서버 설계의 출발점이다. 4주차에서 개념적으로 소개한 내용을 이번 강의에서 구현 관점에서 깊이 있게 다룬다.

#### 6.2.1 도구(Tool): 에이전트가 수행하는 행동

도구는 에이전트가 외부 세계에 영향을 미치는 행동을 정의한다. API를 호출하거나, 데이터베이스에 쿼리를 보내거나, 파일을 생성하는 것이 모두 도구에 해당한다. MCP 서버에서 가장 빈번하게 사용되는 프리미티브이며, 대부분의 서버가 하나 이상의 도구를 등록한다.

각 도구는 네 가지 구성 요소를 가진다.

| 구성 요소 | 역할 | 예시 |
|----------|------|------|
| 이름(name) | LLM이 도구를 식별하는 고유 식별자 | `get_current_weather` |
| 설명(description) | LLM이 도구의 용도를 판단하는 근거 | "지정한 도시의 현재 기온과 날씨 상태를 조회한다" |
| 입력 스키마(input schema) | JSON Schema 형식의 파라미터 정의 | `{ "city": { "type": "string" } }` |
| 핸들러 함수(handler) | 실제 로직을 실행하는 비동기 함수 | `async def get_current_weather(city: str)` |

**표 6.1** 도구의 네 가지 구성 요소

도구를 설계할 때 가장 중요한 점은 LLM이 도구를 선택하는 과정을 이해하는 것이다. LLM은 도구의 이름과 설명만을 읽고 "이 도구를 지금 호출해야 하는가"를 판단한다. 코드를 직접 읽지 않는다. 따라서 이름은 명확하고 서술적이어야 하며, 설명은 도구가 무엇을 하고 어떤 상황에서 사용해야 하는지를 구체적으로 기술해야 한다.

도구 호출의 전체 흐름은 다음과 같다.

```
사용자 요청 → LLM이 도구 목록 검토 → 적절한 도구 선택 → 파라미터 구성
→ MCP 서버로 JSON-RPC 요청 전송 → 핸들러 함수 실행 → 결과 반환
→ LLM이 결과를 자연어로 정리 → 사용자에게 응답
```

**그림 6.1** 도구 호출의 전체 흐름

이 과정에서 LLM이 직접 관여하는 단계는 "도구 선택"과 "파라미터 구성"이다. 나머지는 MCP 프로토콜과 서버가 자동으로 처리한다. 따라서 서버 개발자가 가장 신경 써야 할 부분은 LLM이 올바른 판단을 내릴 수 있도록 도구의 메타데이터(이름, 설명, 스키마)를 정확하게 작성하는 것이다.

#### 6.2.2 리소스(Resource): 에이전트가 읽는 데이터

리소스는 에이전트가 읽기 전용으로 접근할 수 있는 데이터를 정의한다. 파일 내용, 데이터베이스 레코드, 설정 정보, 정적 목록 등이 해당한다. 리소스는 URI(Uniform Resource Identifier) 기반으로 접근하며, `config://settings`이나 `db://users/123`처럼 프로토콜과 경로로 식별한다.

도구가 "행동"이라면 리소스는 "데이터"에 해당한다. 에이전트가 작업에 필요한 맥락 정보를 얻는 통로 역할을 한다. 예를 들어, 날씨 MCP 서버에서 "지원하는 도시 목록"은 리소스로 제공하는 것이 적절하다. 이 목록은 자주 변하지 않고, 에이전트가 도구를 호출하기 전에 참고하는 맥락 정보이기 때문이다.

도구와 리소스의 구분이 처음에는 모호하게 느껴질 수 있다. 판단 기준은 다음과 같다.

| 판단 기준 | 도구(Tool) | 리소스(Resource) |
|----------|-----------|-----------------|
| 외부 상태 변경 여부 | 변경한다 (API 호출, 파일 생성) | 변경하지 않는다 (읽기 전용) |
| 파라미터 의존성 | 입력에 따라 결과가 달라진다 | 고정된 데이터를 반환한다 |
| 호출 주체 | LLM이 자율적으로 호출한다 | 애플리케이션이 명시적으로 요청한다 |
| 예시 | 메시지 전송, 날씨 조회, DB 쿼리 | 설정 파일, 도시 목록, 스키마 정보 |

**표 6.2** 도구와 리소스의 구분 기준

사용자 프로필을 조회하는 것은 리소스에 가깝고, 사용자 프로필을 수정하는 것은 도구에 가깝다. 상태를 변경하는가, 아니면 이미 존재하는 데이터를 그대로 반환하는가가 핵심 기준이다.

#### 6.2.3 프롬프트(Prompt): 재사용 가능한 템플릿

프롬프트는 자주 사용하는 프롬프트 패턴을 서버 측에서 템플릿으로 제공하는 기능이다. 예를 들어 "요약" 프롬프트나 "코드 리뷰" 프롬프트를 미리 정의해 두면, 클라이언트가 이를 호출하여 일관된 형식의 지시를 LLM에 전달할 수 있다.

프롬프트는 인자를 받아 동적으로 내용을 구성할 수 있다는 점에서 단순한 문자열 상수와 다르다. 예를 들어 "코드 리뷰" 프롬프트가 프로그래밍 언어와 리뷰 관점을 인자로 받아 맞춤형 리뷰 지시를 생성할 수 있다.

다만 현재 MCP 생태계에서 프롬프트는 도구나 리소스에 비해 활용 빈도가 낮다. Copilot에서는 Skills(`.github/copilot-instructions.md`)가 유사한 역할을 수행하기 때문이다. 이번 실습에서는 도구와 리소스에 집중하되, 프롬프트의 존재와 용도는 알아 둔다.

#### 6.2.4 세 요소의 종합 구조

다음은 이 세 요소가 하나의 MCP 서버 안에서 어떻게 구성되는지를 보여 주는 구조도이다.

```
MCP Server
├── Tools (행동)
│   ├── get_weather: 날씨 조회
│   └── send_notification: 알림 전송
├── Resources (데이터)
│   └── config://settings: 설정 정보
└── Prompts (템플릿)
    └── summarize: 요약 프롬프트
```

**그림 6.2** MCP 서버의 세 가지 핵심 요소(Primitive)

실제 MCP 서버를 설계할 때는 기능 목록을 먼저 나열한 뒤, 각 기능이 "행동인가, 데이터인가, 템플릿인가"를 판단하여 적절한 프리미티브에 배치하는 순서를 따른다. 이 분류가 명확할수록 LLM이 서버의 기능을 정확하게 이해하고 활용할 수 있다.

### 6.3 Python MCP SDK와 FastMCP 패턴

#### 6.3.1 MCP Python SDK 개요

MCP 서버는 다양한 언어로 구현할 수 있지만, 이 강의에서는 Python SDK를 사용한다. Python SDK는 Anthropic이 공식으로 관리하는 라이브러리로, 서버 구현에 필요한 모든 기능을 제공한다.

SDK의 핵심 구성 요소는 다음과 같다.

| 구성 요소 | 역할 |
|----------|------|
| `FastMCP` 클래스 | 서버 인스턴스를 생성하고 도구/리소스/프롬프트를 등록하는 진입점 |
| 전송 계층(Transport) | STDIO, SSE, Streamable HTTP 등 통신 방식 관리 |
| JSON-RPC 핸들링 | MCP 프로토콜 메시지의 직렬화/역직렬화 자동 처리 |
| 타입 시스템 | Python 타입 힌트를 JSON Schema로 자동 변환 |

**표 6.3** MCP Python SDK의 핵심 구성 요소

`FastMCP`는 SDK가 제공하는 고수준(high-level) API이다. 저수준 API를 사용하면 JSON-RPC 핸들러를 직접 구현해야 하지만, `FastMCP`를 사용하면 데코레이터 한 줄로 도구와 리소스를 등록할 수 있다. Flask가 웹 라우팅을 데코레이터로 간소화한 것과 동일한 철학이다.

#### 6.3.2 데코레이터 패턴의 이해

Python의 데코레이터(Decorator)는 함수를 감싸서 추가 기능을 부여하는 문법이다. `@` 기호로 함수 위에 붙인다. FastMCP는 이 패턴을 활용하여 일반 Python 함수를 MCP 도구나 리소스로 변환한다.

```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("my-server")

@mcp.tool()
async def greet(name: str) -> str:
    """사용자에게 인사한다."""
    return f"안녕하세요, {name}님!"
```

위 코드에서 `@mcp.tool()` 데코레이터가 하는 일은 세 가지이다.

1. 함수 이름(`greet`)을 도구의 이름으로 등록한다
2. 독스트링(`"사용자에게 인사한다."`)을 도구의 설명으로 등록한다
3. 타입 힌트(`name: str`)를 JSON Schema로 변환하여 입력 스키마를 생성한다

개발자가 별도의 JSON Schema를 수동으로 작성할 필요가 없다는 것이 FastMCP 패턴의 핵심적인 장점이다. 함수 시그니처와 독스트링만 잘 작성하면 SDK가 나머지를 자동으로 처리한다.

리소스 등록도 동일한 패턴을 따른다.

```python
@mcp.resource("greeting://templates")
async def get_templates() -> str:
    """인사말 템플릿 목록을 반환한다."""
    return "한국어: 안녕하세요, 영어: Hello, 일본어: こんにちは"
```

`@mcp.resource()` 데코레이터의 인자로 URI를 전달한다. 클라이언트는 이 URI(`greeting://templates`)를 통해 해당 리소스에 접근한다.

#### 6.3.3 타입 힌트와 JSON Schema 자동 변환

FastMCP는 Python의 타입 힌트를 분석하여 JSON Schema를 자동으로 생성한다. 이 변환 규칙을 이해하면 더 정확한 스키마를 설계할 수 있다.

| Python 타입 힌트 | JSON Schema 타입 | 비고 |
|-----------------|-----------------|------|
| `str` | `"type": "string"` | 가장 일반적 |
| `int` | `"type": "integer"` | 정수만 허용 |
| `float` | `"type": "number"` | 실수 허용 |
| `bool` | `"type": "boolean"` | true/false |
| `list[str]` | `"type": "array", "items": {"type": "string"}` | 문자열 배열 |
| `Optional[str]` | `"type": "string"` + 필수 아님 | 선택적 파라미터 |

**표 6.4** Python 타입 힌트와 JSON Schema 변환 규칙

기본값이 있는 파라미터는 선택적(optional)으로 처리된다. 기본값이 없는 파라미터는 필수(required)로 처리된다. 이 규칙을 활용하면 선택적 파라미터와 필수 파라미터를 자연스럽게 구분할 수 있다.

```python
@mcp.tool()
async def search_news(keyword: str, limit: int = 5) -> str:
    """뉴스를 검색한다. keyword는 필수, limit는 선택(기본 5건)."""
    ...
```

위 함수에서 `keyword`는 필수 파라미터이고, `limit`는 기본값이 5인 선택적 파라미터이다. SDK는 이를 자동으로 인식하여 스키마에 반영한다.

### 6.4 도구 설명 작성법: LLM이 읽는 문서

MCP 서버를 만들 때 흔히 저지르는 실수는 코드 로직에만 집중하고 도구의 이름과 설명을 대충 작성하는 것이다. 그러나 LLM은 도구의 소스 코드를 읽지 않는다. LLM이 도구를 선택할 때 참고하는 정보는 오직 세 가지, 즉 도구의 이름, 설명, 그리고 파라미터 스키마뿐이다.

#### 6.4.1 나쁜 예와 좋은 예

설명이 모호하면 LLM은 적절한 상황에서도 도구를 호출하지 못하거나, 잘못된 상황에서 호출할 수 있다. 반대로 설명이 구체적이고 정확하면 LLM의 도구 선택 정확도가 크게 향상된다.

| 구분 | 나쁜 예 | 좋은 예 |
|-----|--------|--------|
| 이름 | `do_stuff` | `get_current_weather` |
| 설명 | "날씨 관련 작업" | "지정한 도시의 현재 기온, 습도, 날씨 상태를 조회한다. 도시명은 한국어 또는 영문으로 입력 가능하다." |
| 파라미터 | `data: str` | `city: str` (description: "조회할 도시 이름 (예: '서울', 'Tokyo')") |

**표 6.5** 도구 설명의 좋은 예와 나쁜 예

좋은 예에서 주목할 점은 세 가지이다. 이름이 동사+명사 형태(`get_current_weather`)로 행동을 명확히 나타내고, 설명이 반환하는 정보의 종류(기온, 습도, 날씨 상태)를 구체적으로 열거하며, 파라미터에 예시 값(`'서울'`, `'Tokyo'`)이 포함되어 LLM이 어떤 형식의 값을 전달해야 하는지 직관적으로 파악할 수 있다.

#### 6.4.2 도구 설명 작성의 네 가지 원칙

첫째, 이름은 `동사_명사` 형태로 작성한다. `get_weather`, `search_news`, `create_report`처럼 도구가 수행하는 행동을 이름만으로 알 수 있어야 한다. `handle`, `process`, `do`와 같은 범용적인 동사는 피한다.

둘째, 설명에는 "무엇을 하는가"와 "언제 사용해야 하는가"를 모두 포함한다. 단순히 "날씨를 조회한다"보다는 "지정한 도시의 현재 기온, 습도, 날씨 상태를 조회한다. 사용자가 특정 지역의 날씨를 물을 때 사용한다."가 더 효과적이다.

셋째, 파라미터마다 설명과 예시를 포함한다. FastMCP에서는 함수의 타입 힌트가 자동으로 JSON Schema로 변환되지만, 파라미터의 의미까지 자동으로 추론되지는 않는다. 독스트링에 각 파라미터의 용도와 예시를 명시하는 습관을 들이는 것이 좋다.

넷째, 도구의 수가 많아질수록 설명의 차별성이 중요하다. 서버에 도구가 3개일 때는 LLM이 비교적 쉽게 올바른 도구를 선택하지만, 10개 이상이 되면 이름과 설명의 차별성이 떨어질수록 잘못된 도구가 선택될 확률이 높아진다. 도구의 이름이 기능을 정확히 반영하고, 설명이 서로 겹치지 않도록 작성하는 것이 대규모 MCP 서버를 운영하는 데 있어 필수적인 습관이다.

#### 6.4.3 독스트링 작성 패턴

FastMCP에서 도구 설명의 품질을 높이는 가장 효과적인 방법은 독스트링을 체계적으로 작성하는 것이다. 다음 패턴을 권장한다.

```python
@mcp.tool()
async def get_weather(city: str, units: str = "metric") -> str:
    """지정한 도시의 현재 기온과 날씨 상태를 조회한다.

    사용자가 특정 도시의 날씨, 기온, 기상 상태를 물을 때 사용한다.
    도시명은 한국어('서울', '부산') 또는 영문('Tokyo', 'New York')으로 입력 가능하다.

    Args:
        city: 조회할 도시 이름. 예: '서울', 'Tokyo'
        units: 온도 단위. 'metric'(섭씨, 기본값) 또는 'imperial'(화씨)
    """
    ...
```

첫 줄은 도구의 핵심 기능을 한 문장으로 요약한다. 그 아래에 사용 상황과 입력 형식을 기술한다. `Args` 섹션에서 각 파라미터의 의미와 예시를 명시한다. 이 패턴을 따르면 SDK가 자동 생성하는 스키마의 품질이 크게 향상된다.

### 6.5 개발 환경 준비

MCP 서버를 만들기 위해 Python 프로젝트를 하나 생성한다. 가상환경을 만들고 필요한 패키지를 설치하는 과정은 다음과 같다.

```bash
mkdir my-mcp-server && cd my-mcp-server
python3 -m venv venv && source venv/bin/activate  # Windows: venv\Scripts\activate
pip install mcp httpx python-dotenv
```

각 패키지의 역할은 다음과 같다.

| 패키지 | 역할 | 비고 |
|-------|------|------|
| `mcp` | MCP 서버 공식 Python SDK | FastMCP, Transport, JSON-RPC 포함 |
| `httpx` | 비동기 HTTP 클라이언트 | `async`/`await` 네이티브 지원 |
| `python-dotenv` | `.env` 파일에서 환경 변수 로드 | API 키 관리용 |

**표 6.6** 필수 패키지와 역할

`mcp` 패키지는 MCP 서버를 구현하기 위한 공식 Python SDK이다. 이 패키지 안에 FastMCP 클래스, 전송 계층(transport), JSON-RPC 핸들링 등 서버 구현에 필요한 모든 기능이 포함되어 있다. `httpx`는 비동기 HTTP 클라이언트로, 외부 API를 호출할 때 사용한다. Python 표준 라이브러리의 `urllib`와 달리 `async`/`await` 구문을 네이티브로 지원하므로 MCP 서버의 비동기 핸들러와 잘 어울린다. `python-dotenv`는 `.env` 파일에서 API 키와 같은 환경 변수를 불러오는 데 활용한다.

프로젝트 폴더 안에 `.env` 파일을 생성하고 API 키를 기록해 둔다. 이 파일은 `.gitignore`에 반드시 추가하여 저장소에 커밋되지 않도록 한다. API 키가 공개 저장소에 노출되면 타인이 악용할 수 있으므로 이 단계는 반드시 지켜야 한다.

```
OPENWEATHER_API_KEY=발급받은_API_키
```

OpenWeatherMap은 무료 플랜으로 분당 60회 호출을 허용하므로 학습 용도로 충분하다. 회원가입 후 API 키를 발급받으면 즉시 사용할 수 있다. 발급 직후에는 활성화까지 몇 분이 걸릴 수 있으므로, 수업 전에 미리 발급해 두는 것을 권장한다.

프로젝트의 최종 폴더 구조는 다음과 같다.

```
my-mcp-server/
├── .env                    # API 키 (Git에 커밋하지 않음)
├── .gitignore              # .env, venv/ 등 제외
├── weather_server.py       # MCP 서버 메인 파일
├── requirements.txt        # 의존성 목록
└── venv/                   # Python 가상환경
```

**그림 6.3** MCP 서버 프로젝트 폴더 구조

---

## 2회차: 실습

### 실습 1: 날씨 API MCP 서버 구현

이 실습에서는 도시 이름을 받아 현재 날씨를 반환하는 도구와, 지원 도시 목록을 반환하는 리소스를 갖춘 MCP 서버를 처음부터 끝까지 구현한다.

#### 단계 1: 프로젝트 생성과 환경 설정

터미널에서 프로젝트를 생성하고 필요한 패키지를 설치한다.

```bash
mkdir my-mcp-server && cd my-mcp-server
python3 -m venv venv && source venv/bin/activate
pip install mcp httpx python-dotenv
```

`.env` 파일을 생성하고 OpenWeatherMap API 키를 입력한다.

```
OPENWEATHER_API_KEY=여기에_발급받은_키_입력
```

`.gitignore` 파일을 생성하여 민감한 파일을 제외한다.

```
.env
venv/
__pycache__/
```

#### 단계 2: 서버 기본 뼈대 작성

`weather_server.py` 파일을 생성하고 서버의 뼈대를 작성한다.

```python
import os
import httpx
from dotenv import load_dotenv
from mcp.server.fastmcp import FastMCP

load_dotenv()

API_KEY = os.getenv("OPENWEATHER_API_KEY")
BASE_URL = "https://api.openweathermap.org/data/2.5/weather"

mcp = FastMCP("weather-server")
```

`FastMCP` 클래스의 인자로 전달하는 문자열은 서버의 이름으로, 클라이언트가 여러 서버를 구분할 때 사용한다. 하나의 VS Code 프로젝트에 여러 MCP 서버를 동시에 연결할 수 있으므로, 서버 이름은 기능을 직관적으로 나타내는 것이 좋다.

#### 단계 3: 도구 등록 - get_weather

날씨 조회 도구를 등록한다. 1회차에서 학습한 도구 설명 작성 원칙을 적용한다.

```python
@mcp.tool()
async def get_weather(city: str) -> str:
    """지정한 도시의 현재 기온과 날씨 상태를 조회한다.

    사용자가 특정 도시의 날씨를 물을 때 사용한다.
    도시명은 한국어('서울') 또는 영문('Tokyo')으로 입력 가능하다.

    Args:
        city: 조회할 도시 이름. 예: '서울', 'Tokyo', '부산'
    """
    if not city.strip():
        return "오류: 도시 이름이 비어 있다. 도시명을 입력해 달라."

    try:
        async with httpx.AsyncClient(timeout=10.0) as client:
            resp = await client.get(
                BASE_URL,
                params={
                    "q": city,
                    "appid": API_KEY,
                    "units": "metric",
                    "lang": "kr",
                },
            )
            resp.raise_for_status()
        data = resp.json()
        temp = data["main"]["temp"]
        humidity = data["main"]["humidity"]
        desc = data["weather"][0]["description"]
        return f"{city}: {temp}°C, 습도 {humidity}%, {desc}"
    except httpx.TimeoutException:
        return f"오류: {city} 날씨 조회 시 타임아웃이 발생했다. 잠시 후 다시 시도해 달라."
    except httpx.HTTPStatusError as e:
        if e.response.status_code == 404:
            return f"오류: '{city}'에 해당하는 도시를 찾을 수 없다. 도시명을 확인해 달라."
        return f"오류: API가 {e.response.status_code} 상태를 반환했다."
```

이 코드에서 주목할 점은 다음과 같다.

- `@mcp.tool()` 데코레이터가 함수를 MCP 도구로 자동 등록한다
- 독스트링이 도구의 설명으로 사용되므로 구체적으로 작성한다
- `city: str` 타입 힌트가 JSON Schema로 자동 변환된다
- 오류 메시지는 LLM이 읽고 판단할 수 있도록 구체적으로 작성한다
- `httpx.AsyncClient`에 `timeout=10.0`을 명시적으로 설정한다

#### 단계 4: 리소스 등록 - 지원 도시 목록

지원하는 도시 목록을 리소스로 등록한다.

```python
@mcp.resource("cities://list")
async def list_cities() -> str:
    """지원하는 도시 목록을 반환한다.

    사용자가 어떤 도시의 날씨를 조회할 수 있는지 알고 싶을 때 참고한다.
    """
    cities = ["서울", "부산", "대구", "인천", "광주", "대전", "울산", "제주"]
    return ", ".join(cities)
```

`@mcp.resource()` 데코레이터의 인자로 URI(`cities://list`)를 전달한다. 이 URI를 통해 클라이언트가 리소스에 접근한다. 도구와 달리 리소스는 읽기 전용 데이터를 제공하므로 외부 상태를 변경하지 않는다.

#### 단계 5: 서버 진입점 추가와 실행

파일의 마지막에 서버 진입점을 추가한다.

```python
if __name__ == "__main__":
    mcp.run(transport="stdio")
```

`transport="stdio"`는 표준 입출력(Standard I/O)을 통해 JSON-RPC 메시지를 주고받는 방식이다. 로컬에서 MCP 서버를 실행할 때 가장 일반적으로 사용하는 전송 방식이며, VS Code Copilot도 이 방식으로 MCP 서버와 통신한다.

### 실습 2: MCP Inspector로 서버 테스트

서버를 만들었다고 바로 Copilot에 연결하기보다는, 먼저 독립적으로 서버가 올바르게 동작하는지 검증하는 것이 좋다. Copilot 환경에서 디버깅하면 LLM의 응답과 도구 오류가 뒤섞여 문제 원인을 파악하기 어렵다.

#### MCP Inspector 실행

MCP Inspector는 서버를 LLM 없이 독립적으로 테스트할 수 있는 공식 도구이다.

```bash
npx @modelcontextprotocol/inspector python weather_server.py
```

이 명령을 실행하면 웹 브라우저에 Inspector UI가 열린다. Inspector에서 수행할 수 있는 작업은 다음과 같다.

| 기능 | 설명 |
|------|------|
| 도구 목록 확인 | 서버에 등록된 모든 도구와 그 스키마 확인 |
| 도구 호출 테스트 | 파라미터를 직접 입력하여 도구 실행 |
| 리소스 조회 | 등록된 리소스 URI 목록 확인과 데이터 조회 |
| JSON-RPC 메시지 확인 | 클라이언트-서버 간 원문 메시지 확인 |

**표 6.7** MCP Inspector의 주요 기능

#### 테스트 시나리오

다음 순서로 테스트를 수행한다.

**테스트 1**: 도구 목록 확인
- Inspector 왼쪽 패널에서 "Tools" 탭을 선택한다
- `get_weather` 도구가 목록에 나타나는지 확인한다
- 도구의 설명과 파라미터 스키마가 독스트링에서 정확히 추출되었는지 확인한다

**테스트 2**: 정상적인 도구 호출
- `get_weather`를 클릭하고 `city` 파라미터에 `"서울"`을 입력한다
- "Run" 버튼을 누르고 날씨 정보가 정상적으로 반환되는지 확인한다
- 하단 "Messages" 패널에서 JSON-RPC 요청과 응답 원문을 확인한다

**테스트 3**: 오류 상황 테스트
- `city`에 빈 문자열 `""`을 입력하여 입력 검증이 동작하는지 확인한다
- 존재하지 않는 도시명(예: `"abcxyz"`)을 입력하여 404 오류 처리를 확인한다

**테스트 4**: 리소스 조회
- "Resources" 탭에서 `cities://list` 리소스를 선택한다
- 도시 목록이 정상적으로 반환되는지 확인한다

모든 테스트를 통과하면 서버가 정상적으로 동작하는 것이 검증된 것이다. 이제 다음 주차(7주차)에서 이 서버를 Copilot에 연결하고, 에러 처리를 강화하며, 프로덕션 수준의 안정성을 확보하는 방법을 학습한다.

---

## 과제

### 과제 제출물

1. **날씨 MCP 서버 코드**: `weather_server.py` 파일을 GitHub 리포지토리에 커밋하여 제출한다
2. **Inspector 테스트 결과**: 다음 네 가지 테스트의 스크린샷을 제출한다
   - 도구 목록 확인 화면
   - `"서울"` 날씨 조회 성공 결과
   - 빈 문자열 입력 시 오류 메시지
   - `cities://list` 리소스 조회 결과
3. **도구 설명 개선 보고서** (선택): 날씨 서버의 도구 설명을 개선하기 전후를 비교하고, 개선 근거를 200자 이내로 작성한다

### 평가 기준

| 항목 | 배점 | 기준 |
|------|:----:|------|
| 서버 코드 완성도 | 40% | 도구와 리소스가 올바르게 등록되어 있는가 |
| 도구 설명 품질 | 20% | 독스트링이 구체적이고 LLM이 이해할 수 있는가 |
| Inspector 테스트 | 30% | 네 가지 시나리오를 모두 테스트했는가 |
| 코드 품질 | 10% | 타입 힌트, 주석, 구조의 적절성 |

---

## 핵심 정리

- MCP 서버의 3요소: Tool(행동), Resource(데이터), Prompt(템플릿)이다. 외부 상태를 변경하면 도구, 읽기 전용이면 리소스로 설계한다
- `mcp` Python SDK의 `FastMCP` 패턴은 데코레이터(`@mcp.tool()`, `@mcp.resource()`) 한 줄로 도구와 리소스를 등록할 수 있게 해 준다
- SDK는 함수의 이름, 독스트링, 타입 힌트를 자동으로 분석하여 도구의 이름, 설명, 입력 스키마를 생성한다
- 도구의 이름과 설명은 LLM의 도구 선택을 직접적으로 결정하므로, 구체적이고 명확하게 작성해야 한다
- MCP Inspector로 서버를 독립 테스트한 뒤 Copilot에 연결하는 순서를 따른다. LLM 없이 먼저 검증해야 디버깅이 용이하다
- `transport="stdio"`는 로컬 MCP 서버의 표준 전송 방식이며, VS Code Copilot이 이 방식으로 서버와 통신한다

---

## 참고 자료

- Anthropic. (2025). Model Context Protocol Specification. *MCP GitHub Repository*. https://github.com/modelcontextprotocol/specification
- Anthropic. (2025). MCP Python SDK. *MCP GitHub Repository*. https://github.com/modelcontextprotocol/python-sdk
- Anthropic. (2025). MCP Inspector. *MCP GitHub Repository*. https://github.com/modelcontextprotocol/inspector
- Microsoft. (2025). Use MCP servers in VS Code. *Visual Studio Code Documentation*. https://code.visualstudio.com/docs/copilot/chat/mcp-servers
- OpenWeatherMap. (2025). Current Weather Data API. https://openweathermap.org/current

---

## 다음 주 예고

이번 주차에서 MCP 서버의 기본 구조를 이해하고 날씨 API 서버를 구현했다면, 7주차에서는 이 서버를 프로덕션 수준으로 발전시킨다. 에러 처리와 입력 검증을 체계적으로 강화하고, 타임아웃/재시도/폴백 패턴을 적용하며, 로깅 전략을 설계한다. 또한 만든 서버를 Copilot에 연결하여 실제로 자연어로 동작하는 것을 확인하고, 배포 시 고려해야 할 사항을 다룬다. 2회차 실습에서는 자유 주제로 커스텀 MCP 서버를 설계하고 구현하는 도전 과제를 수행한다.
