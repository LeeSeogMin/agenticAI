# 7주차. MCP 서버 심화

> **1회차** (강의 90분): 에러 처리와 안정성(입력 검증, 타임아웃/재시도/폴백), 로깅 전략, Copilot 연결, 배포 고려사항
> **2회차** (실습 90분): 커스텀 MCP 서버 설계와 구현 (자유 주제), 에러 시나리오 테스트

---

## 학습목표

1. MCP 서버에서 발생할 수 있는 오류 유형을 분류하고, 각 유형에 적합한 처리 패턴을 적용할 수 있다
2. 타임아웃, 재시도, 폴백 패턴을 구현하여 외부 API 의존 서버의 안정성을 확보할 수 있다
3. 구조화된 로깅을 설계하여 운영 중 문제를 신속하게 진단할 수 있다
4. MCP 서버를 Copilot에 연결하고 자연어로 동작을 검증할 수 있다
5. 자유 주제의 커스텀 MCP 서버를 설계부터 테스트까지 독립적으로 완성할 수 있다

## 선수지식

- 6주차: MCP 서버 구현 기초 (FastMCP 패턴, 도구/리소스 등록, Inspector 테스트)
- Python 예외 처리: `try`/`except`/`finally` 구문
- Python `logging` 모듈 기초
- `.vscode/mcp.json` 설정 방법 (5주차)

---

## 1회차: 강의

### 7.1 에러 처리의 원칙: LLM이 읽는 오류 메시지

외부 API를 호출하는 MCP 서버는 필연적으로 다양한 오류 상황에 직면한다. 네트워크 타임아웃, 잘못된 입력, API 응답 오류, 인증 실패, 속도 제한(rate limiting) 등이 발생할 수 있으며, 이를 적절히 처리하지 않으면 서버가 크래시(crash)하거나 LLM이 의미 없는 에러 스택 트레이스를 받게 된다.

오류 처리에서 가장 핵심적인 원칙은 에러 메시지의 수신자가 사람이 아니라 LLM이라는 점이다. 일반적인 웹 서비스에서는 사용자가 에러 메시지를 읽고 대응하지만, MCP 서버에서는 LLM이 에러 메시지를 읽고 사용자에게 상황을 설명하거나 다른 접근 방식을 시도한다. 따라서 에러 메시지는 무엇이 잘못되었고, 어떻게 하면 해결할 수 있는지를 구체적으로 알려 주어야 한다.

다음은 에러 메시지의 나쁜 예와 좋은 예를 비교한 것이다.

| 상황 | 나쁜 예 | 좋은 예 |
|------|--------|--------|
| 도시명 오류 | `"Error 404"` | `"오류: '서욹'에 해당하는 도시를 찾을 수 없다. 올바른 도시명(예: '서울')을 다시 입력해 달라."` |
| 타임아웃 | `"TimeoutError"` | `"오류: 날씨 API 응답이 10초 내에 도착하지 않았다. 서버가 일시적으로 느릴 수 있으므로 잠시 후 다시 시도해 달라."` |
| 인증 실패 | `"401 Unauthorized"` | `"오류: API 키가 유효하지 않다. 서버 관리자에게 API 키 설정을 확인해 달라고 요청해야 한다."` |

**표 7.1** 에러 메시지의 나쁜 예와 좋은 예

좋은 에러 메시지의 공통점은 세 가지이다. 첫째, 무엇이 잘못되었는지를 명확히 기술한다. 둘째, 가능한 원인을 제시한다. 셋째, 해결 방법 또는 다음 행동을 안내한다. LLM은 이 정보를 읽고 사용자에게 상황을 자연어로 설명하거나, 파라미터를 수정하여 재시도할 수 있다.

또 하나의 중요한 원칙은 예외를 상위로 전파(raise)하기보다는 문자열로 래핑하여 반환하는 것이다. MCP 도구에서 처리되지 않은 예외가 발생하면 서버 프로세스 전체가 비정상 종료될 수 있다. 에러 상황을 문자열 메시지로 변환하여 반환하면, 서버는 계속 실행되고 LLM은 에러 내용을 파악하여 적절히 대응할 수 있다.

### 7.2 입력 검증 (Input Validation)

입력 검증은 도구 핸들러 함수의 첫 번째 방어선이다. LLM이 항상 완벽한 입력을 구성하는 것은 아니기 때문에, 도구 함수는 잘못된 입력에 대해 방어적으로 코딩해야 한다.

#### 7.2.1 검증해야 할 항목

MCP 도구에서 검증해야 할 입력 항목은 다음과 같다.

| 검증 항목 | 설명 | 예시 |
|----------|------|------|
| 빈 값 | 빈 문자열, None, 공백만 있는 문자열 | `city = ""`, `city = "   "` |
| 타입 | 예상과 다른 타입의 값 | 숫자를 기대하는데 문자열이 전달 |
| 범위 | 허용 범위를 벗어나는 값 | `limit = -1`, `page = 99999` |
| 형식 | 기대하는 형식과 다른 값 | 날짜 형식이 `YYYY-MM-DD`가 아닌 경우 |
| 길이 | 지나치게 긴 문자열 | 도시명에 1000자 이상의 문자열 전달 |
| 허용 목록 | 사전 정의된 값만 허용 | `units`에 `"metric"` 또는 `"imperial"`만 허용 |

**표 7.2** MCP 도구에서 검증해야 할 입력 항목

#### 7.2.2 검증 패턴 코드

다음은 체계적인 입력 검증 패턴을 보여 주는 코드이다.

```python
VALID_UNITS = {"metric", "imperial"}
MAX_CITY_LENGTH = 100

@mcp.tool()
async def get_weather(city: str, units: str = "metric") -> str:
    """지정한 도시의 현재 날씨를 조회한다."""
    # 1. 빈 값 검증
    if not city or not city.strip():
        return "오류: 도시 이름이 비어 있다. 도시명을 입력해 달라. 예: '서울', 'Tokyo'"

    # 2. 길이 검증
    city = city.strip()
    if len(city) > MAX_CITY_LENGTH:
        return f"오류: 도시 이름이 너무 길다. {MAX_CITY_LENGTH}자 이내로 입력해 달라."

    # 3. 허용 목록 검증
    if units not in VALID_UNITS:
        return f"오류: units는 {VALID_UNITS} 중 하나여야 한다. 입력된 값: '{units}'"

    # 검증 통과 후 실제 로직 실행
    ...
```

검증 함수를 별도로 분리하면 재사용성이 높아진다. 여러 도구가 동일한 파라미터(예: 도시명)를 받는 경우, 검증 로직을 공유할 수 있다.

```python
def validate_city(city: str) -> str | None:
    """도시명을 검증하고 정제한다. 오류 시 에러 메시지를 반환한다."""
    if not city or not city.strip():
        return "오류: 도시 이름이 비어 있다. 도시명을 입력해 달라."
    city = city.strip()
    if len(city) > MAX_CITY_LENGTH:
        return f"오류: 도시 이름이 너무 길다. {MAX_CITY_LENGTH}자 이내로 입력해 달라."
    return None  # 검증 통과
```

이처럼 검증 함수가 `None`을 반환하면 검증 통과, 문자열을 반환하면 해당 문자열이 에러 메시지가 되는 패턴은 간결하면서도 명확하다.

### 7.3 타임아웃, 재시도, 폴백 패턴

외부 API에 의존하는 MCP 서버는 네트워크 상태에 따라 응답이 느려지거나 실패할 수 있다. 이러한 상황에 대비하여 타임아웃(Timeout), 재시도(Retry), 폴백(Fallback) 세 가지 패턴을 적용한다.

#### 7.3.1 타임아웃 설정

`httpx.AsyncClient`를 생성할 때 `timeout` 파라미터를 명시적으로 설정하는 것이 첫 번째 단계이다. 기본값은 5초이지만, 외부 API의 응답 속도에 따라 적절히 조정해야 한다.

```python
async with httpx.AsyncClient(timeout=10.0) as client:
    resp = await client.get(url, params=params)
```

타임아웃이 너무 짧으면 정상적인 요청도 실패하고, 너무 길면 사용자가 오래 기다려야 한다. 일반적으로 10~15초가 적절한 범위이다. 타임아웃을 세분화할 수도 있다.

```python
timeout = httpx.Timeout(
    connect=5.0,   # 연결 수립 타임아웃
    read=10.0,     # 응답 읽기 타임아웃
    write=5.0,     # 요청 쓰기 타임아웃
    pool=5.0,      # 커넥션 풀 대기 타임아웃
)
async with httpx.AsyncClient(timeout=timeout) as client:
    resp = await client.get(url, params=params)
```

연결 수립(connect)은 빠르게 실패해야 하고, 응답 읽기(read)는 상대적으로 여유를 두는 것이 일반적인 전략이다.

#### 7.3.2 재시도 패턴

일시적인 네트워크 오류는 재시도로 해결할 수 있다. 다만 무조건 재시도하면 이미 과부하된 서버에 더 큰 부담을 줄 수 있으므로, 지수 백오프(Exponential Backoff) 전략을 사용한다.

```python
import asyncio

async def fetch_with_retry(url: str, params: dict, max_retries: int = 3) -> dict:
    """지수 백오프 재시도로 HTTP GET 요청을 수행한다."""
    for attempt in range(max_retries):
        try:
            async with httpx.AsyncClient(timeout=10.0) as client:
                resp = await client.get(url, params=params)
                resp.raise_for_status()
                return resp.json()
        except (httpx.TimeoutException, httpx.HTTPStatusError) as e:
            if attempt == max_retries - 1:
                raise  # 마지막 시도에서도 실패하면 예외 전파
            wait_time = 2 ** attempt  # 1초, 2초, 4초 ...
            await asyncio.sleep(wait_time)
```

지수 백오프의 핵심은 재시도 간격을 점진적으로 늘리는 것이다. 첫 번째 재시도는 1초 후, 두 번째는 2초 후, 세 번째는 4초 후에 시도한다. 이렇게 하면 일시적인 과부하 상황에서 서버가 회복할 시간을 확보할 수 있다.

재시도할 수 있는 오류와 재시도해서는 안 되는 오류를 구분하는 것도 중요하다.

| 재시도 가능한 오류 | 재시도 불가능한 오류 |
|------------------|-------------------|
| 타임아웃 (408, 504) | 인증 실패 (401, 403) |
| 서버 오류 (500, 502, 503) | 잘못된 요청 (400) |
| 네트워크 연결 실패 | 리소스 없음 (404) |
| 속도 제한 (429) | 입력 검증 실패 |

**표 7.3** 재시도 가능 여부에 따른 오류 분류

인증 실패나 입력 오류는 동일한 요청을 반복해도 결과가 달라지지 않으므로 재시도가 무의미하다.

#### 7.3.3 폴백 패턴

폴백(Fallback)은 주요 데이터 소스가 실패할 때 대체 소스로 전환하는 패턴이다. MCP 서버에서는 다음과 같은 폴백 전략을 적용할 수 있다.

```python
@mcp.tool()
async def get_weather(city: str) -> str:
    """도시의 현재 날씨를 조회한다."""
    # 1차: 외부 API 호출 시도
    try:
        result = await fetch_with_retry(BASE_URL, params={"q": city, ...})
        return format_weather(city, result)
    except Exception:
        pass

    # 2차: 캐시된 데이터 확인
    cached = get_cached_weather(city)
    if cached:
        return f"{city}: {cached['temp']}°C (캐시된 데이터, {cached['cached_at']} 기준)"

    # 3차: 최소한의 정보 제공
    return (
        f"오류: {city}의 날씨 정보를 가져올 수 없다. "
        "API 서버에 문제가 있을 수 있다. 잠시 후 다시 시도해 달라."
    )
```

이 코드는 세 단계의 폴백을 구현한다. 먼저 외부 API를 호출하고, 실패하면 캐시된 데이터를 확인하며, 캐시도 없으면 사용자에게 상황을 안내하는 메시지를 반환한다. 핵심은 어떤 상황에서든 의미 있는 응답을 반환하는 것이다.

### 7.4 로깅 전략

MCP 서버는 백그라운드 프로세스로 실행되므로, 문제가 발생했을 때 로그가 유일한 진단 수단인 경우가 많다. 체계적인 로깅 전략을 설계해야 한다.

#### 7.4.1 Python logging 모듈 활용

MCP 서버에서는 `print()` 대신 Python 표준 라이브러리의 `logging` 모듈을 사용한다. STDIO 전송 방식에서 `print()`는 JSON-RPC 메시지와 섞여 프로토콜 오류를 일으킬 수 있기 때문이다.

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(name)s: %(message)s",
    handlers=[logging.FileHandler("mcp_server.log")],
)
logger = logging.getLogger("weather-server")
```

로그 출력을 파일로 지정하면 STDIO 스트림과 분리되어 안전하다. 로그 파일은 디버깅 시 서버의 동작 이력을 추적하는 데 활용한다.

#### 7.4.2 로그 레벨 전략

각 로그 레벨의 사용 기준은 다음과 같다.

| 로그 레벨 | 사용 상황 | 예시 |
|----------|----------|------|
| `DEBUG` | 개발 중 상세한 흐름 추적 | 파라미터 값, API 요청 URL |
| `INFO` | 정상적인 주요 이벤트 | 도구 호출 시작/완료, 서버 시작 |
| `WARNING` | 주의가 필요하지만 동작에 문제없는 상황 | 캐시 미스, 폴백 사용 |
| `ERROR` | 오류가 발생했으나 서버는 계속 동작 | API 호출 실패, 타임아웃 |
| `CRITICAL` | 서버 동작이 불가능한 심각한 오류 | API 키 누락, 설정 파일 없음 |

**표 7.4** 로그 레벨별 사용 기준

#### 7.4.3 도구 호출 로깅 패턴

모든 도구 호출에 대해 시작, 완료, 오류를 일관되게 기록하는 것이 좋다.

```python
@mcp.tool()
async def get_weather(city: str) -> str:
    """도시의 현재 날씨를 조회한다."""
    logger.info("get_weather 호출: city=%s", city)

    error = validate_city(city)
    if error:
        logger.warning("입력 검증 실패: city=%s, error=%s", city, error)
        return error

    try:
        result = await fetch_weather(city)
        logger.info("get_weather 성공: city=%s, temp=%s", city, result["temp"])
        return format_weather(city, result)
    except httpx.TimeoutException:
        logger.error("get_weather 타임아웃: city=%s", city)
        return f"오류: {city} 날씨 조회 시 타임아웃이 발생했다."
    except Exception as e:
        logger.error("get_weather 실패: city=%s, error=%s", city, e, exc_info=True)
        return f"오류: {city} 날씨 조회 중 예기치 않은 오류가 발생했다."
```

`logger.error()`에 `exc_info=True`를 전달하면 스택 트레이스가 로그에 포함되어 디버깅에 유용하다.

### 7.5 Copilot에 연결하기

서버가 Inspector 테스트를 통과했다면, 이제 VS Code Copilot의 Agent Mode에서 이 서버를 사용할 수 있도록 설정한다.

#### 7.5.1 mcp.json 설정

프로젝트 루트에 `.vscode/mcp.json` 파일을 생성하고 다음과 같이 작성한다.

```json
{
  "servers": {
    "weather": {
      "command": "python",
      "args": ["weather_server.py"],
      "cwd": "${workspaceFolder}/my-mcp-server"
    }
  }
}
```

`command`와 `args`는 서버를 실행하는 명령어를 지정한다. `cwd`는 작업 디렉터리로, `${workspaceFolder}`는 VS Code가 현재 열고 있는 폴더의 경로를 가리키는 변수이다. 이 설정을 저장하면 Copilot이 자동으로 서버를 시작하고 사용 가능한 도구 목록을 인식한다.

가상환경을 사용하는 경우 `command`에 가상환경 내 Python 경로를 지정해야 한다.

```json
{
  "servers": {
    "weather": {
      "command": "${workspaceFolder}/my-mcp-server/venv/bin/python",
      "args": ["weather_server.py"],
      "cwd": "${workspaceFolder}/my-mcp-server"
    }
  }
}
```

Windows에서는 `venv/bin/python` 대신 `venv\\Scripts\\python.exe`를 사용한다.

#### 7.5.2 Agent Mode에서 테스트

Copilot Chat을 Agent Mode로 전환한 뒤 자연어로 요청해 본다.

**테스트 1**: "서울의 현재 날씨를 알려줘"
- Copilot은 사용자의 요청을 분석하고, 등록된 도구 목록에서 `get_weather`가 적절하다고 판단한다
- `city: "서울"` 인자와 함께 도구를 호출한다
- 서버가 API를 통해 날씨 정보를 가져오면 Copilot은 결과를 자연어로 정리하여 응답한다

**테스트 2**: "지원하는 도시 목록을 보여줘"
- Copilot은 `cities://list` 리소스를 읽어 도시 목록을 반환한다

**테스트 3**: "도쿄와 뉴욕 날씨를 비교해 줘"
- Copilot이 `get_weather`를 두 번 호출하는지 관찰한다
- 두 도시의 결과를 비교하여 자연어로 정리하는지 확인한다

Copilot이 도구를 호출하기 전에 사용자에게 승인을 요청하는 확인 대화상자가 나타난다. 이것은 보안을 위한 설계이다. MCP 도구는 외부 API를 호출하거나 시스템에 영향을 줄 수 있으므로, 사용자가 매번 실행 여부를 확인하는 것이 기본 동작이다. 세션 내에서 동일한 도구에 대해 반복 승인이 번거롭다면 "이 세션 동안 항상 허용"을 선택할 수 있다.

### 7.6 배포 고려사항

학습 환경에서는 STDIO 전송 방식으로 로컬에서 서버를 실행하지만, 실제 팀 프로젝트나 운영 환경에서는 추가적인 고려가 필요하다.

#### 7.6.1 전송 방식 선택

| 전송 방식 | 적합한 상황 | 장점 | 단점 |
|----------|-----------|------|------|
| STDIO | 로컬 개발, 개인 사용 | 설정 간단, 별도 서버 불필요 | 로컬에서만 사용 가능 |
| SSE | 원격 서버, 팀 공유 | 네트워크를 통해 접근 가능 | 서버 호스팅 필요 |
| Streamable HTTP | 클라우드 배포, 대규모 운영 | 확장성, 표준 HTTP 인프라 활용 | 설정 복잡도 높음 |

**표 7.5** MCP 전송 방식별 비교

이 강의에서는 STDIO에 집중하지만, 팀 프로젝트(13~14주차)에서 SSE 방식을 활용할 수 있다는 점을 알아 둔다.

#### 7.6.2 보안 체크리스트

MCP 서버를 배포할 때 반드시 점검해야 할 보안 항목은 다음과 같다.

1. **API 키 관리**: `.env` 파일은 절대 Git에 커밋하지 않는다. `.gitignore`에 반드시 추가한다
2. **입력 새니타이징(sanitizing)**: SQL 인젝션, 명령어 인젝션 등의 공격을 방지한다. 사용자 입력을 직접 명령어나 쿼리에 삽입하지 않는다
3. **속도 제한**: 도구 호출 빈도를 제한하여 API 비용 폭증을 방지한다
4. **최소 권한 원칙**: 서버에 필요한 최소한의 권한만 부여한다. 읽기만 필요한 서버에 쓰기 권한을 주지 않는다
5. **로그 내 민감 정보 제외**: API 키, 비밀번호 등이 로그에 기록되지 않도록 마스킹한다

#### 7.6.3 멱등성(Idempotency) 설계

도구를 설계할 때 가능한 한 멱등성을 유지하는 것이 중요하다. 같은 입력으로 여러 번 호출해도 동일한 결과가 나오도록 설계하면 재시도가 안전해진다. LLM은 네트워크 오류 등으로 도구 호출이 실패했다고 판단하면 동일한 호출을 다시 시도할 수 있기 때문이다.

날씨 조회(`get_weather`)는 본질적으로 멱등적이다. 같은 도시를 여러 번 조회해도 부작용이 없다. 반면, 메시지 전송이나 주문 생성 같은 도구는 멱등적이지 않으므로, 중복 실행을 방지하는 별도의 메커니즘(예: 요청 ID)이 필요할 수 있다.

### 7.7 종합: 프로덕션 수준의 도구 구조

지금까지 학습한 입력 검증, 타임아웃, 재시도, 폴백, 로깅을 모두 결합한 도구의 전체 구조는 다음과 같다.

```python
@mcp.tool()
async def get_weather(city: str) -> str:
    """지정한 도시의 현재 기온과 날씨 상태를 조회한다."""
    logger.info("get_weather 호출: city=%s", city)

    # 1단계: 입력 검증
    error = validate_city(city)
    if error:
        logger.warning("입력 검증 실패: %s", error)
        return error

    # 2단계: 재시도 포함 API 호출
    try:
        data = await fetch_with_retry(
            BASE_URL,
            params={"q": city.strip(), "appid": API_KEY, "units": "metric", "lang": "kr"},
            max_retries=3,
        )
        result = format_weather(city, data)
        logger.info("get_weather 성공: city=%s", city)
        return result

    except httpx.TimeoutException:
        logger.error("get_weather 타임아웃: city=%s", city)
        # 3단계: 폴백 - 캐시 확인
        cached = get_cached_weather(city)
        if cached:
            logger.info("캐시 폴백 사용: city=%s", city)
            return f"{city}: {cached['temp']}°C (캐시 데이터, {cached['cached_at']} 기준)"
        return f"오류: {city} 날씨 조회 시 타임아웃이 발생했다. 잠시 후 다시 시도해 달라."

    except httpx.HTTPStatusError as e:
        logger.error("API 오류: city=%s, status=%d", city, e.response.status_code)
        if e.response.status_code == 404:
            return f"오류: '{city}'에 해당하는 도시를 찾을 수 없다. 도시명을 확인해 달라."
        return f"오류: API가 {e.response.status_code} 상태를 반환했다."

    except Exception as e:
        logger.error("예기치 않은 오류: city=%s, error=%s", city, e, exc_info=True)
        return f"오류: {city} 날씨 조회 중 예기치 않은 오류가 발생했다."
```

이 구조는 다섯 가지 방어 계층을 갖추고 있다.

1. **입력 검증**: 잘못된 입력을 API 호출 전에 차단한다
2. **타임아웃**: 응답이 지연되는 경우 적절한 시간 내에 실패를 판정한다
3. **재시도**: 일시적인 오류에 대해 지수 백오프로 재시도한다
4. **폴백**: 재시도도 실패하면 캐시 데이터로 대체한다
5. **포괄적 예외 처리**: 예상하지 못한 오류도 서버를 종료시키지 않고 안전하게 처리한다

---

## 2회차: 실습

### 실습 1: 날씨 서버 안정성 강화

6주차에서 구현한 날씨 MCP 서버에 1회차에서 학습한 패턴들을 적용하여 안정성을 강화한다.

#### 단계 1: 입력 검증 함수 추가

`weather_server.py`에 검증 함수를 추가한다.

```python
MAX_CITY_LENGTH = 100

def validate_city(city: str) -> str | None:
    """도시명을 검증한다. 오류 시 에러 메시지, 정상 시 None을 반환한다."""
    if not city or not city.strip():
        return "오류: 도시 이름이 비어 있다. 도시명을 입력해 달라. 예: '서울', 'Tokyo'"
    if len(city.strip()) > MAX_CITY_LENGTH:
        return f"오류: 도시 이름이 너무 길다. {MAX_CITY_LENGTH}자 이내로 입력해 달라."
    return None
```

#### 단계 2: 재시도 함수 추가

지수 백오프 재시도 함수를 구현한다.

```python
import asyncio

async def fetch_with_retry(url: str, params: dict, max_retries: int = 3) -> dict:
    """지수 백오프 재시도로 HTTP GET 요청을 수행한다."""
    for attempt in range(max_retries):
        try:
            async with httpx.AsyncClient(timeout=10.0) as client:
                resp = await client.get(url, params=params)
                resp.raise_for_status()
                return resp.json()
        except (httpx.TimeoutException, httpx.HTTPStatusError) as e:
            if attempt == max_retries - 1:
                raise
            wait_time = 2 ** attempt
            logger.warning("재시도 %d/%d: %s초 대기", attempt + 1, max_retries, wait_time)
            await asyncio.sleep(wait_time)
```

#### 단계 3: 로깅 설정 추가

파일 상단에 로깅 설정을 추가한다.

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(name)s: %(message)s",
    handlers=[logging.FileHandler("weather_server.log")],
)
logger = logging.getLogger("weather-server")
```

#### 단계 4: get_weather 도구 리팩토링

기존 `get_weather` 함수를 1회차에서 학습한 종합 구조(7.7절)로 리팩토링한다.

#### 단계 5: Inspector + Copilot 테스트

먼저 Inspector로 다음 시나리오를 테스트한다.

| 시나리오 | 입력 | 기대 결과 |
|---------|------|----------|
| 정상 호출 | `city: "서울"` | 기온, 습도, 날씨 상태 반환 |
| 빈 문자열 | `city: ""` | 입력 검증 에러 메시지 |
| 존재하지 않는 도시 | `city: "abcxyz"` | 404 에러 메시지 |
| 긴 문자열 | `city: "a" * 200` | 길이 검증 에러 메시지 |

**표 7.6** 에러 시나리오 테스트 목록

Inspector 테스트 통과 후, `.vscode/mcp.json`에 서버를 등록하고 Copilot Agent Mode에서 자연어 테스트를 수행한다.

### 실습 2: 커스텀 MCP 서버 도전 과제

이번 강의에서 배운 내용을 종합하여, 자신에게 의미 있는 도메인의 MCP 서버를 직접 설계하고 구현한다. 날씨 서버가 외부 API를 래핑하는 패턴이었다면, 이번에는 자신이 선택한 도메인에서 설계부터 시작하는 것이 과제의 요점이다. 실제 API가 없다면 모의 데이터(mock data)를 사용해도 좋다.

#### 추천 주제와 요구사항

다음 주제 중 하나를 선택하거나, 자유 주제로 진행한다.

**주제 A: 학교 정보 MCP 서버**

구현할 요소는 세 가지 도구와 한 가지 리소스이다.

| 유형 | 이름 | 기능 |
|------|------|------|
| 도구 | `get_menu` | 오늘의 학식 메뉴 조회 (날짜 파라미터, 기본값: 오늘) |
| 도구 | `get_room_schedule` | 특정 강의실 사용 현황 조회 (건물명, 호수) |
| 도구 | `get_notice` | 학교 공지사항 키워드 검색 |
| 리소스 | `campus://info` | 학교 기본 정보 (이름, 주소, 학과 목록) |

**표 7.7** 학교 정보 MCP 서버 구성

학식 메뉴는 매일 바뀌고 날짜 파라미터에 따라 다른 결과를 반환하므로 도구가 적절하다. 반면 학교 기본 정보는 거의 변하지 않는 고정 데이터이므로 리소스로 제공하는 것이 자연스럽다.

**주제 B: 도서관 검색 MCP 서버**

| 유형 | 이름 | 기능 |
|------|------|------|
| 도구 | `search_books` | 제목/저자로 도서 검색 |
| 도구 | `check_availability` | 특정 도서의 대출 가능 여부 확인 |
| 도구 | `get_popular_books` | 이번 달 인기 도서 목록 조회 |
| 리소스 | `library://info` | 도서관 운영 시간, 위치, 연락처 |

**주제 C: 자유 주제**

자신이 자주 사용하는 서비스나 데이터를 MCP 서버로 래핑한다. 도구 2개 이상, 리소스 1개 이상을 포함해야 한다.

#### 공통 요구사항

주제에 관계없이 다음 요구사항을 충족해야 한다.

1. **도구 2개 이상 + 리소스 1개 이상** 구현
2. **도구 설명**: 독스트링에 기능, 사용 상황, 파라미터 예시를 포함
3. **입력 검증**: 모든 도구에 빈 값, 범위, 타입 검증 포함
4. **에러 처리**: 타임아웃, HTTP 오류, 예기치 않은 오류에 대한 처리
5. **로깅**: 도구 호출 시작/완료/오류를 로그에 기록
6. **Inspector 테스트**: 정상 시나리오 + 에러 시나리오 각 2건 이상 테스트
7. **Copilot 연결**: `.vscode/mcp.json` 설정 후 Agent Mode에서 자연어 테스트

#### 구현 순서 가이드

1. 기능 목록을 작성하고 도구/리소스를 분류한다 (10분)
2. 프로젝트 구조를 만들고 의존성을 설치한다 (5분)
3. 서버 뼈대를 작성한다: FastMCP 인스턴스 생성 (5분)
4. 모의 데이터를 준비한다 (10분)
5. 도구를 하나씩 구현하고, 구현할 때마다 Inspector로 테스트한다 (30분)
6. 리소스를 구현한다 (10분)
7. 입력 검증과 에러 처리를 추가한다 (10분)
8. `.vscode/mcp.json`에 등록하고 Copilot에서 테스트한다 (10분)

---

## 과제

### 과제 제출물

1. **커스텀 MCP 서버 코드**: 서버 메인 파일을 GitHub 리포지토리에 커밋하여 제출한다
2. **설계 문서**: 도구/리소스 분류 근거를 간단히 기술한다 (200자 이내)
3. **테스트 결과**: 다음 항목의 스크린샷을 제출한다
   - Inspector에서 정상 호출 결과 2건
   - Inspector에서 에러 처리 결과 2건
   - Copilot Agent Mode에서 자연어 테스트 결과 1건
4. **로그 파일**: 서버 동작 중 생성된 로그 파일을 제출한다

### 평가 기준

| 항목 | 배점 | 기준 |
|------|:----:|------|
| 서버 설계 | 20% | 도구/리소스 분류가 합리적인가 |
| 도구 설명 품질 | 15% | 독스트링이 LLM 친화적인가 |
| 입력 검증 | 15% | 모든 도구에 적절한 검증이 포함되었는가 |
| 에러 처리 | 20% | 타임아웃, HTTP 오류, 예외를 적절히 처리하는가 |
| 로깅 | 10% | 호출 시작/완료/오류가 기록되는가 |
| 테스트 완성도 | 15% | 정상 + 에러 시나리오를 모두 테스트했는가 |
| Copilot 연결 | 5% | Agent Mode에서 정상 동작하는가 |

---

## 핵심 정리

- MCP 서버의 에러 메시지는 LLM이 읽고 판단하는 텍스트이다. 원인과 해결 방법을 함께 제공해야 한다
- 입력 검증은 도구 핸들러의 첫 번째 방어선이다. 빈 값, 범위, 길이, 허용 목록을 확인한다
- 외부 API 의존 서버는 타임아웃(10~15초), 재시도(지수 백오프), 폴백(캐시/기본값) 세 가지 패턴을 적용한다
- 재시도 가능한 오류(타임아웃, 서버 오류)와 재시도 불가능한 오류(인증 실패, 404)를 구분해야 한다
- `print()` 대신 `logging` 모듈을 사용한다. STDIO 전송에서 `print()`는 프로토콜 오류를 유발할 수 있다
- Copilot 연결 전에 반드시 Inspector로 독립 테스트를 완료한다
- 멱등성을 유지하면 LLM의 자동 재시도가 안전해진다
- API 키는 `.env`로 관리하고, `.gitignore`에 반드시 추가한다

---

## 참고 자료

- Anthropic. (2025). Model Context Protocol Specification. *MCP GitHub Repository*. https://github.com/modelcontextprotocol/specification
- Anthropic. (2025). MCP Python SDK. *MCP GitHub Repository*. https://github.com/modelcontextprotocol/python-sdk
- Anthropic. (2025). MCP Inspector. *MCP GitHub Repository*. https://github.com/modelcontextprotocol/inspector
- Microsoft. (2025). Use MCP servers in VS Code. *Visual Studio Code Documentation*. https://code.visualstudio.com/docs/copilot/chat/mcp-servers
- Encode. (2025). HTTPX - Async HTTP Client. https://www.python-httpx.org/
- Python Software Foundation. (2025). logging - Logging facility for Python. *Python Documentation*. https://docs.python.org/3/library/logging.html

---

## 다음 주 예고

6~7주차에서 MCP가 Copilot에 "도구"를 제공하는 방법을 학습했다. 8주차 중간고사를 거친 뒤, 9주차에서는 Skills(Custom Instructions)로 Copilot에 "지식과 절차"를 가르치는 방법을 다룬다. MCP가 Copilot의 손과 발이라면, Skills는 Copilot의 머릿속에 도메인 전문성을 심어 주는 것이다. 예를 들어 "우리 팀의 코드 리뷰 기준에 맞춰 검토해 줘"라는 요청을 Copilot이 이해하려면, 그 기준이 무엇인지를 미리 가르쳐야 한다. 9주차에서는 이를 SKILL.md 파일로 구현하는 방법을 실습한다. 중간고사 범위는 1~7주차 전체이므로, 복습을 충분히 해 두는 것을 권장한다.
