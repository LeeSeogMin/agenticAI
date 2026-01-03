# 제3장: MCP 개념과 도구/리소스 설계

## 학습목표

1. Model Context Protocol(MCP)의 개념과 필요성을 이해할 수 있다
2. MCP의 도구(tool)와 리소스(resource) 설계 원칙을 적용할 수 있다
3. 최소 MCP 서버의 구조와 통신 방식(STDIO)을 설명할 수 있다
4. MCP 도구의 입출력, 에러 처리, 제약 조건을 설계할 수 있다

---

## 3.1 왜 MCP가 필요한가

AI 코딩 어시스턴트를 사용하다 보면 반복되는 패턴을 발견하게 된다. "이 디렉토리의 파일 목록을 보여줘", "이 API를 호출해서 결과를 파일로 저장해줘", "이 데이터베이스에서 특정 조건의 레코드를 가져와줘" 같은 요청이다. 이런 요청은 단순해 보이지만, AI가 매번 처음부터 코드를 생성하면 일관성이 떨어지고 오류가 발생할 가능성도 높아진다.

더 큰 문제는 AI가 외부 시스템에 접근할 때 발생한다. 파일 시스템, 데이터베이스, 웹 API, 클라우드 서비스 등 다양한 외부 자원에 안전하고 일관된 방식으로 접근하려면 표준화된 인터페이스가 필요하다. 각 AI 도구(ChatGPT, Claude, GitHub Copilot)마다 다른 방식으로 외부 자원에 접근한다면, 코드를 재사용하거나 이식하기 어렵다.

### 3.1.1 AI 도구 통합의 문제점

현재 AI 코딩 어시스턴트들은 각자 다른 방식으로 외부 도구와 통합한다. ChatGPT는 플러그인(Plugins) 또는 GPTs를 통해 외부 API를 호출하고, Claude는 Tool Use API를 제공하며, GitHub Copilot은 VS Code의 확장 기능과 통합된다. 각 플랫폼마다 API 형식, 인증 방식, 오류 처리 방법이 다르므로, 하나의 도구를 여러 플랫폼에서 사용하려면 플랫폼별로 별도의 어댑터를 구현해야 한다.

예를 들어, Notion API를 호출하는 도구를 만든다고 하자. ChatGPT 플러그인으로 만들면 OpenAPI 스펙을 작성하고, OAuth 인증을 구현하고, 플러그인 매니페스트를 작성해야 한다. 동일한 기능을 Claude에서 사용하려면 Anthropic의 Tool Use API 형식에 맞춰 함수 정의를 다시 작성해야 한다. GitHub Copilot에서 사용하려면 VS Code 확장으로 패키징해야 한다. 같은 기능을 세 가지 다른 방식으로 구현하는 것은 비효율적이다.

도구 간 연결도 문제다. AI 워크플로우에서는 여러 도구를 조합하여 복잡한 작업을 수행한다. "GitHub에서 이슈 목록을 가져와서, 각 이슈의 내용을 요약하고, 요약본을 Notion 데이터베이스에 저장한다"는 작업은 GitHub API, LLM 호출, Notion API 세 가지 도구를 순차적으로 사용한다. 각 도구가 서로 다른 인터페이스를 사용하면, 도구 간 데이터를 전달하고 오류를 처리하는 코드가 복잡해진다.

보안과 권한 관리도 중요하다. AI가 파일 시스템에 접근하거나 외부 API를 호출할 때, 어떤 작업은 허용하고 어떤 작업은 차단할지 제어해야 한다. 예를 들어, AI가 파일을 읽는 것은 허용하지만 삭제는 허용하지 않거나, 특정 디렉토리 밖에는 접근하지 못하게 해야 할 수 있다. 각 AI 플랫폼마다 권한 모델이 다르면, 동일한 보안 정책을 여러 플랫폼에 일관되게 적용하기 어렵다.

### 3.1.2 Model Context Protocol의 등장

Model Context Protocol(MCP)은 Anthropic이 2024년 11월에 발표한 표준 프로토콜로, AI 모델과 외부 데이터/도구 간의 통신 방식을 통일한다. MCP는 "AI가 어떻게 외부 컨텍스트에 접근하는가"에 대한 공통 언어를 제공한다. 웹 개발에서 HTTP가 클라이언트와 서버 간 통신 표준이 된 것처럼, MCP는 AI와 외부 자원 간 통신 표준이 되려는 시도다.

MCP의 핵심 아이디어는 "서버-클라이언트 아키텍처"다. 외부 자원(파일 시스템, 데이터베이스, API 등)에 접근하는 로직은 **MCP 서버**로 구현한다. AI 애플리케이션(ChatGPT, Claude Desktop, 커스텀 에이전트 등)은 **MCP 클라이언트**가 되어, 표준화된 프로토콜을 통해 MCP 서버와 통신한다. 하나의 MCP 서버를 구현하면, 여러 AI 클라이언트에서 동일하게 사용할 수 있다.

MCP는 세 가지 핵심 개념을 정의한다:

**첫째, 도구(Tools)**: AI가 호출할 수 있는 함수다. 도구는 명확한 이름, 설명, 입력 스키마, 출력 형식을 가진다. 예를 들어, `read_file` 도구는 "파일 경로를 입력받아 파일 내용을 반환한다"는 기능을 제공한다. AI는 도구의 설명을 읽고 적절한 시점에 도구를 호출한다.

**둘째, 리소스(Resources)**: AI가 읽을 수 있는 컨텍스트 데이터다. 리소스는 도구와 달리 호출하지 않고 "조회"한다. 예를 들어, 프로젝트의 README 파일, 데이터베이스 스키마, API 문서 등이 리소스가 될 수 있다. AI는 리소스를 읽어 현재 상황을 이해하고 더 나은 판단을 내린다.

**셋째, 프롬프트(Prompts)**: 재사용 가능한 프롬프트 템플릿이다. 특정 작업에 자주 사용되는 프롬프트를 MCP 서버에 등록하면, AI 클라이언트에서 쉽게 불러와 사용할 수 있다. 예를 들어, "코드 리뷰 프롬프트", "버그 분석 프롬프트" 같은 템플릿을 서버에 두고, 필요할 때마다 가져다 쓸 수 있다.

MCP는 통신 방식으로 **JSON-RPC 2.0**을 사용한다. JSON-RPC는 경량 원격 프로시저 호출(RPC) 프로토콜로, HTTP나 WebSocket 위에서 동작할 수 있다. MCP 서버와 클라이언트는 JSON-RPC 메시지를 주고받으며 통신한다. 표준 프로토콜을 사용하므로, 다양한 언어와 플랫폼에서 MCP를 구현할 수 있다.

### 3.1.3 MCP의 장점과 제약

MCP를 사용하면 다음과 같은 장점이 있다:

**플랫폼 독립성**: 하나의 MCP 서버를 만들면, Claude Desktop, ChatGPT, 커스텀 에이전트 등 여러 클라이언트에서 동일하게 사용할 수 있다. 플랫폼마다 별도의 어댑터를 만들 필요가 없다.

**재사용성**: 자주 사용하는 기능(파일 읽기, 데이터베이스 조회, API 호출 등)을 MCP 서버로 구현하면, 여러 프로젝트에서 재사용할 수 있다. 한 번 잘 만든 MCP 서버는 여러 팀과 프로젝트에서 공유할 수 있는 자산이 된다.

**보안 경계**: MCP 서버는 독립적인 프로세스로 실행되므로, 권한과 접근 제어를 명확히 할 수 있다. AI가 파일 시스템에 무제한 접근하는 대신, MCP 서버가 허용된 작업만 수행하도록 제한할 수 있다. 예를 들어, 읽기 전용 파일 서버, 특정 디렉토리만 접근 가능한 서버 등을 만들 수 있다.

**테스트 가능성**: MCP 서버는 독립적인 컴포넌트이므로, AI 없이도 단독으로 테스트할 수 있다. 서버의 입출력이 명확히 정의되어 있으므로, 단위 테스트를 작성하기 쉽다. AI 에이전트 전체를 실행하지 않고도 특정 도구의 동작을 검증할 수 있다.

**명확한 인터페이스**: 도구와 리소스의 스키마가 명시적으로 정의되므로, AI가 어떤 기능을 사용할 수 있는지, 각 기능이 무엇을 하는지 명확히 알 수 있다. 이는 AI의 도구 선택 정확도를 높이고, 디버깅을 쉽게 만든다.

하지만 MCP에도 제약이 있다:

**초기 학습 곡선**: MCP 서버를 만들려면 JSON-RPC 프로토콜, 스키마 정의, 비동기 통신 등 새로운 개념을 배워야 한다. 간단한 스크립트를 작성하는 것보다 초기 투자 비용이 크다.

**오버헤드**: MCP는 서버-클라이언트 구조이므로, 프로세스 간 통신(IPC) 오버헤드가 있다. 매우 빈번히 호출되는 간단한 함수는 직접 호출하는 것보다 느릴 수 있다. 하지만 대부분의 AI 워크플로우에서는 네트워크 I/O나 LLM 호출 시간이 훨씬 크므로, MCP 오버헤드는 무시할 수 있는 수준이다.

**표준 미성숙**: MCP는 2024년 11월에 발표된 비교적 새로운 프로토콜이다. 2026년 현재도 활발히 개발 중이며, 일부 기능이 불안정하거나 문서가 부족할 수 있다. 프로덕션 환경에서 사용하려면 안정성과 호환성을 면밀히 검토해야 한다.

**클라이언트 지원**: 모든 AI 플랫폼이 MCP를 지원하는 것은 아니다. Claude Desktop은 MCP를 네이티브로 지원하지만, 다른 플랫폼은 별도의 통합 작업이 필요할 수 있다. 커스텀 에이전트에서는 MCP 클라이언트 라이브러리를 사용하면 되지만, 상용 플랫폼의 지원 여부는 확인해야 한다.

이런 제약에도 불구하고, MCP는 AI 도구 생태계를 표준화하는 강력한 도구다. 특히 팀이나 조직 내에서 여러 AI 프로젝트를 진행할 때, MCP를 사용하면 도구를 일관되게 관리하고 재사용할 수 있어 효율성이 크게 향상된다.

---

## 3.2 MCP 서버의 기본 구조

MCP 서버는 "AI가 호출할 수 있는 기능을 제공하는 프로그램"이다. 서버는 표준 입출력(STDIO) 또는 HTTP를 통해 클라이언트와 통신한다. 로컬 환경에서는 STDIO 방식이 간단하고 효율적이므로, 이 장에서는 STDIO 기반 MCP 서버를 중심으로 설명한다.

### 3.2.1 STDIO 통신 방식

STDIO(Standard Input/Output) 통신은 프로세스 간 통신(IPC)의 가장 간단한 형태다. MCP 클라이언트는 서버 프로세스를 실행하고, 표준 입력(stdin)으로 요청을 보내고, 표준 출력(stdout)에서 응답을 받는다. 서버는 stdin에서 JSON-RPC 요청을 읽고, 처리 결과를 stdout으로 출력한다.

이 방식의 장점은 다음과 같다:

**간단한 설정**: 네트워크 포트를 열거나 HTTP 서버를 구성할 필요 없이, 단순히 프로세스를 실행하면 된다. 방화벽이나 네트워크 설정에 영향받지 않는다.

**보안**: 프로세스는 클라이언트의 권한으로 실행되므로, 서버가 수행할 수 있는 작업이 자동으로 제한된다. 별도의 인증이나 권한 부여 메커니즘이 필요 없다.

**로컬 최적화**: 프로세스 간 통신은 네트워크 통신보다 빠르고 안정적이다. 로컬 개발 환경에서 사용하기에 적합하다.

STDIO 통신의 동작 흐름은 다음과 같다:

1. 클라이언트가 MCP 서버 프로세스를 실행한다 (예: `python3 mcp_server.py`)
2. 클라이언트가 서버의 stdin으로 JSON-RPC 요청을 전송한다
3. 서버가 stdin에서 요청을 읽고 파싱한다
4. 서버가 요청을 처리하고 결과를 생성한다
5. 서버가 stdout으로 JSON-RPC 응답을 출력한다
6. 클라이언트가 stdout에서 응답을 읽고 파싱한다

각 메시지는 한 줄의 JSON으로 표현되며, 줄바꿈 문자(`\n`)로 구분된다. 이를 "줄 단위 JSON(line-delimited JSON)"이라고 한다. 예를 들어:

```json
{"jsonrpc":"2.0","method":"tools/list","id":1}
{"jsonrpc":"2.0","result":{"tools":[...]},"id":1}
```

첫 줄은 클라이언트의 요청(도구 목록 조회), 두 번째 줄은 서버의 응답(도구 목록)이다.

### 3.2.2 JSON-RPC 2.0 프로토콜

MCP는 JSON-RPC 2.0 프로토콜을 사용한다. JSON-RPC는 JSON으로 인코딩된 원격 프로시저 호출 프로토콜로, 간단하면서도 강력하다. JSON-RPC 2.0 스펙은 https://www.jsonrpc.org/specification 에서 확인할 수 있다.

JSON-RPC 요청은 다음 필드를 포함한다:

- `jsonrpc`: 프로토콜 버전, 항상 `"2.0"`
- `method`: 호출할 메서드 이름 (예: `"tools/list"`, `"tools/call"`)
- `params`: 메서드에 전달할 매개변수 (객체 또는 배열)
- `id`: 요청 식별자 (숫자 또는 문자열, 응답과 매칭하기 위해 사용)

JSON-RPC 응답은 다음 필드를 포함한다:

- `jsonrpc`: 프로토콜 버전, 항상 `"2.0"`
- `result`: 성공 시 결과 (오류 시 생략)
- `error`: 오류 시 오류 정보 (성공 시 생략)
- `id`: 요청의 `id`와 동일한 값

예를 들어, 도구 목록을 조회하는 요청과 응답은 다음과 같다:

```json
// 요청
{
  "jsonrpc": "2.0",
  "method": "tools/list",
  "id": 1
}

// 응답
{
  "jsonrpc": "2.0",
  "result": {
    "tools": [
      {
        "name": "read_file",
        "description": "파일의 내용을 읽습니다",
        "inputSchema": {
          "type": "object",
          "properties": {
            "path": {"type": "string", "description": "읽을 파일의 경로"}
          },
          "required": ["path"]
        }
      }
    ]
  },
  "id": 1
}
```

오류가 발생한 경우 응답은 다음과 같다:

```json
{
  "jsonrpc": "2.0",
  "error": {
    "code": -32602,
    "message": "Invalid params",
    "data": "Missing required parameter: path"
  },
  "id": 1
}
```

JSON-RPC 2.0은 표준 오류 코드를 정의한다:

- `-32700`: Parse error (JSON 파싱 실패)
- `-32600`: Invalid Request (요청 형식 오류)
- `-32601`: Method not found (메서드 없음)
- `-32602`: Invalid params (매개변수 오류)
- `-32603`: Internal error (내부 오류)

애플리케이션 정의 오류는 `-32000 ~ -32099` 범위를 사용한다.

### 3.2.3 MCP 서버의 생명주기

MCP 서버는 다음과 같은 생명주기를 가진다:

**1. 초기화 단계**: 클라이언트가 서버를 실행하고, `initialize` 메서드를 호출하여 서버의 기능(지원하는 도구, 리소스, 프롬프트)을 확인한다. 서버는 자신이 제공하는 기능 목록을 반환한다.

```python
# initialize 요청 처리
def handle_initialize(params):
    return {
        "protocolVersion": "2024-11-05",
        "serverInfo": {
            "name": "example-mcp-server",
            "version": "1.0.0"
        },
        "capabilities": {
            "tools": {},
            "resources": {}
        }
    }
```

_전체 코드는 practice/chapter3/code/3-2-minimal-server.py 참고_

**2. 작업 단계**: 클라이언트가 도구를 호출하거나 리소스를 조회한다. 서버는 요청을 처리하고 결과를 반환한다. 이 단계가 반복된다.

**3. 종료 단계**: 클라이언트가 작업을 완료하면, 서버 프로세스를 종료한다. 서버는 리소스를 정리하고 프로세스를 종료한다.

서버는 상태를 유지할 수 있다. 예를 들어, 데이터베이스 연결을 초기화 단계에서 생성하고, 작업 단계에서 재사용하고, 종료 단계에서 닫을 수 있다. 하지만 가능하면 상태를 최소화하는 것이 좋다. 무상태(stateless) 서버는 테스트와 디버깅이 쉽고, 멀티 클라이언트 환경에서도 안전하다.

### 3.2.4 최소 MCP 서버 구현

가장 간단한 MCP 서버는 하나의 도구만 제공한다. 아래는 "현재 시각을 반환하는 도구"를 제공하는 최소 서버다:

```python
import json
import sys
from datetime import datetime

def handle_tools_list():
    return {
        "tools": [{
            "name": "get_current_time",
            "description": "현재 시각을 반환합니다",
            "inputSchema": {"type": "object", "properties": {}}
        }]
    }
```

_전체 코드는 practice/chapter3/code/3-2-minimal-server.py 참고_

이 서버는 stdin에서 JSON-RPC 요청을 읽고, 메서드에 따라 적절한 핸들러를 호출하고, 결과를 stdout으로 출력한다. 실제 프로덕션 환경에서는 오류 처리, 로깅, 입력 검증 등을 추가해야 하지만, 핵심 구조는 이와 같다.

---

## 3.3 도구(Tool) 설계 원칙

MCP 도구는 "AI가 호출할 수 있는 함수"다. 도구를 잘 설계하려면 명확한 입출력, 적절한 에러 처리, 합리적인 제약 조건이 필요하다.

### 3.3.1 도구의 구성 요소

MCP 도구는 다음 요소로 구성된다:

**이름(name)**: 도구를 식별하는 고유한 문자열이다. 이름은 간결하면서도 기능을 명확히 표현해야 한다. 관례적으로 `snake_case`를 사용한다 (예: `read_file`, `search_database`, `send_email`).

**설명(description)**: 도구가 무엇을 하는지 자연어로 설명한다. AI는 이 설명을 읽고 적절한 도구를 선택한다. 설명은 명확하고 구체적이어야 한다. "파일을 읽습니다"보다 "지정된 경로의 파일 내용을 UTF-8 인코딩으로 읽어 문자열로 반환합니다"가 더 좋다.

**입력 스키마(inputSchema)**: 도구가 받는 매개변수의 타입과 제약을 정의한다. JSON Schema 형식을 사용한다. 예를 들어:

```json
{
  "type": "object",
  "properties": {
    "path": {
      "type": "string",
      "description": "읽을 파일의 절대 또는 상대 경로"
    },
    "encoding": {
      "type": "string",
      "description": "파일 인코딩 (기본값: utf-8)",
      "default": "utf-8"
    }
  },
  "required": ["path"]
}
```

이 스키마는 `path`가 필수 매개변수이고, `encoding`은 선택적 매개변수임을 나타낸다.

**출력 형식**: MCP 도구의 출력은 항상 JSON 직렬화 가능한 값이어야 한다. 문자열, 숫자, 배열, 객체 등을 반환할 수 있다. 복잡한 객체는 명확한 구조를 가져야 한다.

### 3.3.2 명확한 입출력 설계

도구의 입출력은 AI와 개발자 모두에게 명확해야 한다. 다음 원칙을 따르자:

**단일 책임**: 하나의 도구는 하나의 명확한 작업을 수행한다. "파일을 읽고 요약하고 저장한다"는 너무 많은 책임이다. 대신 `read_file`, `summarize_text`, `write_file` 세 개의 도구로 분리한다.

**타입 명시**: 매개변수와 반환값의 타입을 명확히 정의한다. JSON Schema를 최대한 활용하여 문자열 패턴(`pattern`), 숫자 범위(`minimum`, `maximum`), 배열 길이(`minItems`, `maxItems`) 등을 제약한다.

**기본값 제공**: 선택적 매개변수에는 합리적인 기본값을 제공한다. 예를 들어, 파일 인코딩은 `"utf-8"`을 기본값으로 한다. 기본값을 문서화하여 AI가 생략할 수 있음을 알린다.

**예시 포함**: 도구 설명에 사용 예시를 포함하면 AI가 올바르게 사용할 가능성이 높아진다. 예를 들어:

```json
{
  "name": "search_files",
  "description": "파일명 패턴으로 파일을 검색합니다. 예: pattern='*.py'는 모든 Python 파일을 찾습니다.",
  "inputSchema": {...}
}
```

**오류 정보**: 오류 발생 시 구체적인 정보를 제공한다. "Error"보다 "File not found: /path/to/file.txt"가 훨씬 유용하다. 오류 메시지는 AI가 문제를 이해하고 수정할 수 있을 만큼 구체적이어야 한다.

### 3.3.3 에러 처리 전략

MCP 도구는 다양한 오류 상황을 만날 수 있다. 파일이 없거나, 권한이 없거나, 네트워크가 끊기거나, 입력이 잘못되었을 수 있다. 에러를 잘 처리하면 AI가 문제를 자동으로 해결하거나 사용자에게 명확한 안내를 제공할 수 있다.

**입력 검증**: 도구를 실행하기 전에 입력을 검증한다. JSON Schema가 타입을 검증하지만, 비즈니스 로직 검증은 도구 코드에서 수행해야 한다. 예를 들어, 파일 경로가 허용된 디렉토리 내에 있는지, 파일 크기가 제한을 초과하지 않는지 등을 확인한다.

```python
def read_file(path: str) -> str:
    if not path.startswith("/allowed/dir"):
        raise ValueError("Access denied")
    if not os.path.exists(path):
        raise FileNotFoundError(f"File not found: {path}")
    return open(path, "r", encoding="utf-8").read()
```

_전체 코드는 practice/chapter3/code/3-3-tool-validation.py 참고_

**명확한 오류 메시지**: 오류 메시지는 무엇이 잘못되었고, 어떻게 고치면 되는지 명시한다. "Invalid input"보다 "Parameter 'count' must be between 1 and 100, got 150"이 훨씬 유용하다.

**오류 분류**: 오류를 복구 가능/불가능으로 분류한다. 일시적 네트워크 오류는 재시도로 해결할 수 있지만, 권한 오류는 재시도해도 소용없다. 오류 코드나 타입으로 이를 구분하면, AI가 적절한 대응을 할 수 있다.

```python
class RetryableError(Exception):
    """재시도 가능한 오류"""
    pass

class PermanentError(Exception):
    """재시도 불가능한 오류"""
    pass
```

**부분 실패 처리**: 배치 작업에서 일부 항목이 실패할 수 있다. 이 경우 성공한 항목과 실패한 항목을 구분하여 반환한다.

```python
{
  "processed": 8,
  "failed": 2,
  "errors": [{"item": "file3.txt", "error": "Permission denied"}]
}
```

_전체 코드는 practice/chapter3/code/3-3-tool-validation.py 참고_

이렇게 하면 AI는 성공한 부분을 활용하고, 실패한 부분만 다시 시도할 수 있다.

### 3.3.4 제약 조건 설계

MCP 도구는 무제한으로 동작하면 안 된다. 적절한 제약을 설정하여 안전성과 성능을 보장해야 한다.

**리소스 제한**: 메모리, 시간, 파일 크기 등에 제한을 둔다. 예를 들어, 파일 읽기 도구는 최대 10MB까지만 읽도록 제한한다. 큰 파일은 스트리밍이나 청크 단위 읽기를 사용한다.

```python
MAX_FILE_SIZE = 10 * 1024 * 1024  # 10MB

def read_file(path: str) -> str:
    if os.path.getsize(path) > MAX_FILE_SIZE:
        raise ValueError(f"File too large")
    return open(path, "r").read()
```

_전체 코드는 practice/chapter3/code/3-3-tool-validation.py 참고_

**접근 제어**: 도구가 접근할 수 있는 범위를 제한한다. 파일 시스템 도구는 특정 디렉토리 내부만 접근하게 하고, 데이터베이스 도구는 읽기 전용 권한만 사용한다.

**타임아웃**: 외부 시스템을 호출하는 도구는 타임아웃을 설정한다. 네트워크 요청, 데이터베이스 쿼리, 외부 프로세스 실행 등은 무한정 대기하지 않도록 한다.

```python
import requests

def call_api(url: str) -> dict:
    response = requests.get(url, timeout=10)
    response.raise_for_status()
    return response.json()
```

_전체 코드는 practice/chapter3/code/3-3-tool-validation.py 참고_

**속도 제한**: API 호출이나 비용이 드는 작업은 속도 제한을 적용한다. 예를 들어, 외부 API를 호출하는 도구는 분당 최대 60회로 제한할 수 있다.

**검증 규칙**: 입력이 위험한 패턴을 포함하지 않는지 검증한다. SQL 쿼리 도구는 DROP, DELETE 같은 위험한 키워드를 차단하고, 파일 경로는 `..`(상위 디렉토리 접근)를 금지할 수 있다.

---

## 3.4 리소스(Resource) 설계

리소스는 AI가 읽을 수 있는 컨텍스트 데이터다. 도구와 달리 "실행"하지 않고 "조회"한다. 리소스는 AI가 현재 상황을 이해하고 더 나은 판단을 내리는 데 도움을 준다.

### 3.4.1 리소스의 역할

리소스는 다음과 같은 용도로 사용된다:

**문서 제공**: 프로젝트의 README, API 문서, 데이터베이스 스키마 등을 리소스로 제공하면, AI가 프로젝트 구조를 이해하고 적절한 코드를 생성할 수 있다.

**상태 공유**: 현재 작업의 상태, 설정, 환경 정보를 리소스로 노출하면, AI가 컨텍스트를 유지하면서 작업을 진행할 수 있다. 예를 들어, 현재 Git 브랜치, 환경 변수, 실행 로그 등이 리소스가 될 수 있다.

**데이터 접근**: 데이터베이스 레코드, 파일 내용, API 응답 등을 리소스로 제공하면, AI가 도구를 호출하지 않고도 데이터를 참조할 수 있다. 읽기 전용 접근이 필요한 경우 리소스가 적합하다.

**템플릿 제공**: 자주 사용하는 코드 템플릿, 설정 파일 예시, 프롬프트 템플릿 등을 리소스로 제공하면, AI가 일관된 형식을 따를 수 있다.

리소스는 도구보다 경량이다. 도구는 실행 시마다 계산이나 I/O가 발생하지만, 리소스는 미리 준비된 데이터를 단순히 반환한다. 자주 참조되는 정보는 리소스로 제공하는 것이 효율적이다.

### 3.4.2 리소스 스키마 정의

MCP 리소스는 다음 필드로 정의된다:

**uri**: 리소스를 식별하는 URI다. 관례적으로 `<프로토콜>://<경로>` 형식을 사용한다. 예: `file:///project/README.md`, `db://schema/users`, `config://settings`.

**name**: 사람이 읽을 수 있는 리소스 이름 (선택적).

**description**: 리소스가 무엇을 포함하는지 설명 (선택적).

**mimeType**: 리소스 내용의 MIME 타입 (예: `text/plain`, `application/json`).

리소스 목록을 조회하는 요청과 응답 예시:

```json
// 요청
{
  "jsonrpc": "2.0",
  "method": "resources/list",
  "id": 2
}

// 응답
{
  "jsonrpc": "2.0",
  "result": {
    "resources": [
      {
        "uri": "file:///project/README.md",
        "name": "Project README",
        "mimeType": "text/markdown"
      },
      {
        "uri": "config://database",
        "name": "Database Configuration",
        "mimeType": "application/json"
      }
    ]
  },
  "id": 2
}
```

특정 리소스의 내용을 읽는 요청과 응답:

```json
// 요청
{
  "jsonrpc": "2.0",
  "method": "resources/read",
  "params": {
    "uri": "file:///project/README.md"
  },
  "id": 3
}

// 응답
{
  "jsonrpc": "2.0",
  "result": {
    "contents": [
      {
        "uri": "file:///project/README.md",
        "mimeType": "text/markdown",
        "text": "# My Project\n\nThis is a sample project..."
      }
    ]
  },
  "id": 3
}
```

### 3.4.3 정적 리소스 vs 동적 리소스

리소스는 정적이거나 동적일 수 있다.

**정적 리소스**: 내용이 거의 변하지 않는 리소스다. README 파일, API 문서, 설정 템플릿 등이 해당한다. 정적 리소스는 서버 시작 시 한 번 읽고 캐시할 수 있다.

**동적 리소스**: 실행 시점마다 내용이 달라지는 리소스다. 현재 시각, 로그 파일, 데이터베이스 상태 등이 해당한다. 동적 리소스는 매번 조회 시 최신 값을 반환한다.

동적 리소스는 도구와 비슷해 보이지만, 차이가 있다. 도구는 "작업을 수행"하고 리소스는 "정보를 제공"한다. 예를 들어, `get_current_time`은 도구가 아니라 리소스로 구현하는 것이 적합하다. 시각을 "가져오는" 것은 작업이 아니라 정보 조회이기 때문이다.

```python
# 리소스로 구현
resources = {
    "time://current": lambda: datetime.now().isoformat()
}

def handle_resources_read(uri):
    if uri in resources:
        content = resources[uri]()
        return {"text": content, "mimeType": "text/plain"}
    else:
        raise ValueError(f"Resource not found: {uri}")
```

_전체 코드는 practice/chapter3/code/3-4-resource-server.py 참고_

### 3.4.4 리소스 vs 도구 선택 기준

언제 리소스를 쓰고 언제 도구를 쓸까? 다음 기준을 참고하자:

**읽기 전용이면 리소스**: 데이터를 읽기만 하고 수정하지 않으면 리소스가 적합하다. 파일 읽기, 데이터베이스 조회(SELECT), 로그 확인 등은 리소스로 구현할 수 있다.

**부작용이 있으면 도구**: 외부 상태를 변경하거나 비용이 드는 작업은 도구로 구현한다. 파일 쓰기, 데이터베이스 변경(INSERT/UPDATE/DELETE), API 호출, 이메일 발송 등은 도구다.

**빈번한 참조는 리소스**: AI가 자주 참조하는 정보는 리소스로 제공한다. 매번 도구를 호출하는 오버헤드를 줄일 수 있다.

**복잡한 로직은 도구**: 여러 단계의 처리가 필요하거나 매개변수가 많으면 도구가 적합하다. 리소스는 간단한 조회에 사용한다.

---

## 3.5 실습: 최소 MCP 서버 구현

이제 배운 내용을 바탕으로 최소 MCP 서버를 구현해보자. 이 서버는 두 가지 도구와 하나의 리소스를 제공한다.

### 3.5.1 요구사항 정의

우리가 만들 MCP 서버는 다음 기능을 제공한다:

**도구 1: `add_numbers`** - 두 숫자를 더한다
- 입력: `a` (number), `b` (number)
- 출력: `result` (number)
- 제약: 입력은 -1000 ~ 1000 범위

**도구 2: `write_file`** - 파일에 텍스트를 쓴다
- 입력: `path` (string), `content` (string)
- 출력: `path` (string), `size` (number)
- 제약: `output/` 디렉토리 내부만 쓰기 가능, 최대 1MB

**리소스: `project://info`** - 프로젝트 정보를 반환한다
- 내용: 서버 이름, 버전, 실행 시각

### 3.5.2 서버 구조 설계

서버는 다음 구조를 가진다:

```
practice/chapter3/
├── code/
│   ├── 3-5-minimal-mcp-server.py  # 메인 서버
│   ├── requirements.txt            # 의존성 (없음, 표준 라이브러리만)
│   └── README.md                   # 실행 방법
├── data/
│   └── output/                     # 파일 쓰기 출력 디렉토리
└── docs/
    └── server-design.md            # 설계 문서
```

설계 문서(`server-design.md`)에는 각 도구의 입출력, 제약 조건, 오류 시나리오를 명시한다.

### 3.5.3 핵심 코드 구현

서버의 핵심 로직은 다음과 같다:

```python
def handle_tools_call(name, arguments):
    if name == "add_numbers":
        a, b = arguments["a"], arguments["b"]
        if not (-1000 <= a <= 1000 and -1000 <= b <= 1000):
            raise ValueError("Numbers must be in range -1000 to 1000")
        return {"result": a + b}
    
    elif name == "write_file":
        path = arguments["path"]
        if not path.startswith("output/"):
            raise ValueError("Can only write to output/ directory")
        # 파일 쓰기 로직...
```

_전체 코드는 practice/chapter3/code/3-5-minimal-mcp-server.py 참고_

### 3.5.4 테스트 방법

MCP 서버는 stdin/stdout으로 통신하므로, 간단히 테스트할 수 있다:

```bash
cd practice/chapter3
python3 -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
python3 code/3-5-minimal-mcp-server.py
```

서버가 실행되면, 수동으로 JSON-RPC 요청을 입력하고 응답을 확인할 수 있다:

```json
{"jsonrpc":"2.0","method":"tools/list","id":1}
```

엔터를 누르면 서버가 도구 목록을 출력한다. 실제 클라이언트(Claude Desktop, 커스텀 스크립트)와 통합하려면 MCP 클라이언트 라이브러리를 사용한다.

### 3.5.5 설계 문서 작성

`docs/server-design.md`에는 다음 내용을 포함한다:

- 서버 목적과 제공 기능
- 각 도구의 상세 스펙 (입출력, 제약, 오류 코드)
- 각 리소스의 URI와 내용
- 사용 예시와 예상 시나리오
- 제한 사항과 향후 개선 방향

설계 문서는 팀원이나 미래의 자신이 서버를 이해하는 데 필수적이다. AI 협업 시에도 설계 문서를 리소스로 제공하면, AI가 서버를 올바르게 사용할 수 있다.

---

## 핵심정리

- **MCP(Model Context Protocol)**: AI와 외부 도구/데이터 간 표준화된 통신 프로토콜
- **도구(Tool)**: AI가 호출할 수 있는 함수, 명확한 입출력과 제약 조건을 가져야 함
- **리소스(Resource)**: AI가 참조할 수 있는 컨텍스트 데이터, 읽기 전용 정보 제공
- **STDIO 통신**: 로컬 환경에서 간단하고 안전한 프로세스 간 통신 방식
- **JSON-RPC 2.0**: MCP가 사용하는 표준 원격 프로시저 호출 프로토콜
- **설계 원칙**: 단일 책임, 명확한 타입, 적절한 제약, 구체적인 오류 메시지

---

## 실습 파일 요약

| 파일 | 경로 | 설명 |
|:---|:---|:---|
| 최소 서버 | practice/chapter3/code/3-2-minimal-server.py | 현재 시각 반환 도구를 제공하는 최소 MCP 서버 |
| 입력 검증 | practice/chapter3/code/3-3-tool-validation.py | 파일 읽기 도구의 입력 검증 예시 |
| 리소스 서버 | practice/chapter3/code/3-4-resource-server.py | 정적/동적 리소스 제공 예시 |
| 완전한 서버 | practice/chapter3/code/3-5-minimal-mcp-server.py | 도구 2개 + 리소스 1개를 제공하는 완전한 서버 |
| 설계 문서 | practice/chapter3/docs/server-design.md | MCP 서버 설계 문서 템플릿 |

---

## 참고문헌

Anthropic. (2024). Model Context Protocol Documentation. https://modelcontextprotocol.io/

JSON-RPC Working Group. (2010). JSON-RPC 2.0 Specification. https://www.jsonrpc.org/specification

Python Software Foundation. (2024). json — JSON encoder and decoder. Python 3.13 Documentation. https://docs.python.org/3/library/json.html
