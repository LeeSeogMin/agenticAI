# 4주차. MCP 개념과 아키텍처

> **1회차** (강의 90분): MCP의 탄생 배경, 클라이언트-서버 아키텍처, 전송 방식, 3대 프리미티브, 생태계 현황
> **2회차** (실습 90분): MCP 생태계 탐색, `.vscode/mcp.json` 기본 설정, Filesystem MCP 서버 연결과 동작 확인

---

## 학습목표

1. MCP(Model Context Protocol)의 탄생 배경과 해결하려는 문제를 설명할 수 있다
2. 호스트-클라이언트-서버 3계층 아키텍처의 역할과 데이터 흐름을 도식화할 수 있다
3. Tool, Resource, Prompt 세 가지 프리미티브의 차이를 구분하고 적절한 용도를 판단할 수 있다
4. `.vscode/mcp.json` 파일을 작성하여 MCP 서버를 VS Code에 등록할 수 있다

## 선수지식

- VS Code 기본 사용법 (파일 생성, 터미널 실행, 확장 프로그램 설치)
- Copilot Agent Mode의 동작 원리 (3주차 내용)
- JSON 파일의 기본 문법 (키-값 쌍, 중첩 객체, 배열)
- Node.js와 `npx` 명령의 기본 개념

---

## 1회차: 강의

### 4.1 Agent Mode의 한계와 MCP의 필요성

3주차까지 학습한 Copilot Agent Mode는 코드를 자율적으로 작성하고, 터미널 명령을 실행하며, 파일을 편집하는 강력한 기능을 제공한다. 그러나 Agent Mode만으로는 다음과 같은 작업을 수행할 수 없다.

- 웹 브라우저를 열어 특정 사이트의 정보를 수집하는 작업
- GitHub API를 호출하여 이슈를 생성하거나 PR을 관리하는 작업
- 외부 데이터베이스에 접속하여 데이터를 조회하는 작업
- Slack이나 Notion 같은 협업 도구와 연동하는 작업

이 한계는 Agent Mode가 VS Code 내부의 파일 시스템과 터미널에만 접근할 수 있기 때문에 발생한다. 외부 세계와 상호작용하려면 별도의 연결 통로가 필요하다. 이 문제를 해결하기 위해 등장한 것이 모델 컨텍스트 프로토콜(Model Context Protocol, MCP)이다.

MCP는 AI 에이전트와 외부 도구를 연결하는 범용 표준 프로토콜이다. 흔히 "AI 세계의 USB-C"라고 비유된다. USB-C가 하나의 포트로 충전, 데이터 전송, 영상 출력을 모두 처리하듯이, MCP는 하나의 프로토콜로 AI 에이전트가 다양한 외부 시스템과 상호작용할 수 있게 한다. MCP 이전에는 각 AI 도구가 외부 서비스마다 개별 연동 코드를 작성해야 했다. N개의 AI 도구와 M개의 외부 서비스가 있으면 N x M개의 연동이 필요했던 것이다. MCP는 이를 N + M으로 줄여 준다. 각 AI 도구는 MCP 클라이언트 하나만 구현하고, 각 외부 서비스는 MCP 서버 하나만 구현하면 모든 조합이 자동으로 연결된다.

#### MCP 이전과 이후의 비교

MCP가 없는 세계에서는 Copilot이 GitHub과 연동하려면 GitHub 전용 플러그인이 필요하고, Cursor가 GitHub과 연동하려면 또 다른 전용 플러그인이 필요하다. 도구 5개와 서비스 10개가 있으면 50개의 개별 연동을 만들어야 한다. MCP가 있는 세계에서는 GitHub MCP 서버 하나만 존재하면 Copilot, Cursor, Claude Desktop, Windsurf 어디서든 동일하게 사용할 수 있다. 5 + 10 = 15개의 구현만으로 모든 조합이 가능해진다.

### 4.2 탄생과 거버넌스

MCP는 2024년 11월 Anthropic이 처음 공개하였다. Anthropic은 자사의 AI 모델 Claude가 외부 도구와 상호작용할 수 있는 표준화된 방법이 필요하다고 판단하여 이 프로토콜을 설계하고 오픈소스로 공개하였다.

공개 이후 MCP는 빠르게 산업 전반으로 확산되었다. OpenAI는 2025년 3월 Agents SDK에 MCP 지원을 추가하였고, Google은 ADK(Agent Development Kit)에서 MCP 서버를 도구로 활용할 수 있도록 하였다. Microsoft는 VS Code와 GitHub Copilot에 MCP 지원을 내장하였다. 이처럼 경쟁 관계에 있는 주요 AI 플랫폼이 모두 MCP를 채택하면서, MCP는 사실상의 업계 표준(de facto standard)이 되었다.

2025년 12월에는 MCP의 거버넌스가 중요한 전환점을 맞이하였다. Anthropic, Block(구 Square), OpenAI가 공동으로 설립한 리눅스 재단 산하 AAIF(Agentic AI Foundation)로 프로토콜 관리 권한이 이관된 것이다. 이 결정의 의미는 크다. MCP가 더 이상 특정 기업의 독점 기술이 아니라, Linux나 Kubernetes처럼 커뮤니티가 주도하는 개방형 산업 표준으로 자리매김하였음을 의미하기 때문이다. 현재 안정 버전은 2025-11-25 스펙이다.

#### AAIF의 역할

AAIF는 MCP 스펙의 발전 방향을 결정하고, 호환성 테스트를 관리하며, 생태계의 건전한 성장을 지원한다. 특정 기업이 프로토콜을 자사에 유리한 방향으로 변경하는 것을 방지하고, 모든 참여자가 동등한 발언권을 가지도록 거버넌스 구조를 설계하였다. 이는 웹 표준을 관리하는 W3C나, 컨테이너 표준을 관리하는 OCI(Open Container Initiative)와 유사한 모델이다.

### 4.3 클라이언트-서버 아키텍처

MCP는 호스트(Host), 클라이언트(Client), 서버(Server)의 3계층 구조로 동작한다. 각 계층의 역할을 명확히 이해하는 것이 중요하다.

**호스트(Host)** 는 사용자가 직접 상호작용하는 애플리케이션이다. VS Code, Claude Desktop, Cursor, Windsurf 등이 호스트에 해당한다. 호스트는 사용자의 입력을 받아 AI 모델에 전달하고, 모델의 응답을 사용자에게 보여주는 역할을 한다.

**클라이언트(Client)** 는 호스트 내부에서 동작하며, MCP 서버와의 통신을 담당한다. 하나의 호스트 안에 여러 MCP 클라이언트가 존재할 수 있으며, 각 클라이언트는 하나의 MCP 서버와 1:1로 연결된다. 클라이언트는 서버가 제공하는 도구 목록을 조회하고, AI 모델의 요청에 따라 특정 도구를 호출하며, 그 결과를 모델에 반환한다.

**서버(Server)** 는 실제 외부 기능을 제공하는 프로그램이다. Playwright MCP 서버는 웹 브라우저 조작 기능을, GitHub MCP 서버는 GitHub API 접근 기능을, Memory MCP 서버는 정보 영속 저장 기능을 제공한다. 서버는 자신이 제공하는 기능을 표준화된 형식으로 클라이언트에 알려주고, 클라이언트의 호출을 받아 실제 작업을 수행한다.

```
┌─────────────────────────────────────────┐
│  호스트 (VS Code + Copilot)              │
│                                         │
│  ┌──────────┐  ┌──────────┐             │
│  │ MCP      │  │ MCP      │             │
│  │ Client A │  │ Client B │   ...       │
│  └────┬─────┘  └────┬─────┘             │
└───────┼──────────────┼──────────────────┘
        │              │
   ┌────▼─────┐   ┌────▼─────┐
   │ MCP      │   │ MCP      │
   │ Server A │   │ Server B │
   │(Playwright)  │(GitHub)  │
   └──────────┘   └──────────┘
```

**그림 4.1** MCP의 호스트-클라이언트-서버 3계층 아키텍처

#### 통신 흐름

실제 통신이 이루어지는 순서는 다음과 같다. 사용자가 호스트(VS Code)에서 "이 저장소의 이슈 목록을 보여줘"라고 입력하면, AI 모델은 이 요청을 분석하여 GitHub MCP 서버의 `list_issues` 도구를 호출해야 한다고 판단한다. 모델은 MCP 클라이언트에 도구 호출을 요청하고, 클라이언트는 해당 MCP 서버에 JSON-RPC 메시지를 전송한다. 서버는 GitHub API를 호출하여 결과를 얻고, 이를 클라이언트에 반환한다. 클라이언트는 결과를 모델에 전달하고, 모델은 결과를 사람이 읽기 쉬운 형태로 가공하여 사용자에게 보여준다.

이 과정에서 핵심적인 원칙이 하나 있다. 도구 실행 전에 반드시 사용자의 승인을 받는다는 것이다. AI 모델이 스스로 판단하여 외부 시스템을 변경하는 것은 위험할 수 있으므로, 호스트는 도구 호출 전에 사용자에게 확인 대화상자를 보여준다. 사용자는 개별 승인, 세션 내 자동 승인, 또는 거부를 선택할 수 있다.

### 4.4 전송 방식: STDIO, SSE, Streamable HTTP

MCP 서버와 클라이언트 간의 통신은 세 가지 전송(transport) 방식을 지원한다. 각 방식은 서로 다른 사용 시나리오에 적합하다.

#### STDIO (Standard Input/Output)

로컬 실행에 가장 적합한 방식이다. 호스트가 MCP 서버 프로세스를 직접 실행하고, 표준 입력(stdin)과 표준 출력(stdout)을 통해 JSON-RPC 메시지를 교환한다. 네트워크 설정이 필요 없고, 서버 프로세스가 호스트와 동일한 머신에서 실행되므로 보안 설정이 간단하다. 이번 강의에서 사용하는 대부분의 MCP 서버가 이 방식을 사용한다.

**표 4.1** STDIO 방식의 장단점

| 장점 | 단점 |
|------|------|
| 설정이 간단하다 | 로컬 머신에서만 동작한다 |
| 네트워크 설정 불필요 | 서버를 별도 배포할 수 없다 |
| 지연 시간이 매우 짧다 | 여러 사용자가 공유할 수 없다 |
| 방화벽 문제 없음 | 서버 장애 시 호스트가 재시작해야 한다 |

#### SSE (Server-Sent Events)

원격 서버와 통신할 때 사용하는 방식이다. HTTP 기반으로 동작하며, 서버에서 클라이언트로의 실시간 이벤트 스트리밍을 지원한다. 클라이언트가 HTTP POST로 요청을 보내면, 서버는 SSE 연결을 통해 결과를 스트리밍으로 반환한다. MCP 초기 버전에서 도입된 원격 통신 방식이다.

#### Streamable HTTP

2025-11-25 스펙에서 새롭게 도입된 전송 방식이다. SSE의 한계를 보완하여 설계되었으며, 단일 HTTP 엔드포인트(`/mcp`)로 요청과 응답을 모두 처리한다. 서버가 빠르게 응답할 수 있는 경우에는 일반 HTTP 응답을 반환하고, 시간이 오래 걸리는 작업에는 SSE로 전환하여 스트리밍한다. OAuth 2.1 인증을 표준으로 지원하므로, 프로덕션 환경에서 원격 MCP 서버를 운영할 때 권장되는 방식이다.

**표 4.2** 세 가지 전송 방식 비교

| 특성 | STDIO | SSE | Streamable HTTP |
|------|-------|-----|-----------------|
| 실행 위치 | 로컬 | 원격 | 원격 |
| 프로토콜 | stdin/stdout | HTTP + SSE | HTTP (단일 엔드포인트) |
| 인증 | 불필요 (로컬) | 커스텀 | OAuth 2.1 표준 |
| 스트리밍 | 지원 | 서버→클라이언트만 | 양방향 |
| 적합한 상황 | 개인 개발 환경 | 레거시 원격 서버 | 프로덕션 원격 서버 |
| 이번 강의 사용 | O | X | X |

### 4.5 세 가지 프리미티브: Tool, Resource, Prompt

MCP 서버는 세 가지 유형의 기능(프리미티브)을 AI 에이전트에 노출할 수 있다. 각 프리미티브는 서로 다른 목적과 사용 패턴을 가진다.

#### Tool (도구)

도구는 AI가 호출할 수 있는 동작(action)이다. 웹 페이지를 탐색하거나, 이슈를 생성하거나, 파일을 변환하는 등의 작업이 도구에 해당한다. 도구는 MCP에서 가장 널리 사용되는 프리미티브이며, AI 에이전트가 "무엇을 할 수 있는가"를 정의한다. 도구 호출은 부작용(side effect)을 일으킬 수 있으므로 반드시 사용자의 승인을 받아 실행된다.

도구는 입력 스키마(JSON Schema)와 출력 형식을 명확히 정의해야 한다. AI 모델은 도구의 이름과 설명(description)을 읽고 언제 어떤 도구를 호출할지 판단하므로, 도구 설명을 잘 작성하는 것이 매우 중요하다. 이 부분은 6주차에서 MCP 서버를 직접 구현할 때 자세히 다룬다.

#### Resource (리소스)

리소스는 AI가 읽을 수 있는 데이터 소스이다. 데이터베이스 테이블, 파일 내용, API 응답 등 읽기 전용 데이터를 제공한다. 도구와 달리 리소스는 시스템 상태를 변경하지 않으므로, 별도의 사용자 승인 없이도 AI가 자유롭게 접근할 수 있다. 리소스는 URI 형식으로 식별되며, `file:///path/to/document.md`나 `db://users/schema` 같은 형태를 가진다.

#### Prompt (프롬프트)

프롬프트는 재사용 가능한 프롬프트 템플릿이다. 특정 작업에 최적화된 프롬프트를 서버 측에서 제공하는 기능이다. 코드 리뷰 체크리스트, 보고서 양식, 분석 절차 등을 프롬프트 템플릿으로 만들어 두면, 사용자가 매번 상세한 지시를 작성하지 않아도 일관된 결과를 얻을 수 있다. 프롬프트는 사용자가 명시적으로 선택하여 사용하며, AI가 자동으로 호출하지는 않는다.

**표 4.3** 세 가지 프리미티브 비교

| 특성 | Tool (도구) | Resource (리소스) | Prompt (프롬프트) |
|------|------------|------------------|------------------|
| 성격 | 동작 (action) | 데이터 (data) | 템플릿 (template) |
| 호출 주체 | AI 모델이 판단 | AI 모델 또는 사용자 | 사용자가 선택 |
| 부작용 | 있을 수 있음 | 없음 (읽기 전용) | 없음 |
| 사용자 승인 | 필수 | 불필요 | 선택 시 |
| 예시 | 이슈 생성, 웹 탐색 | DB 스키마, 파일 내용 | 코드 리뷰 체크리스트 |

#### 의사결정 포인트: 기능을 어떤 프리미티브로 노출할 것인가

MCP 서버를 설계할 때, 특정 기능을 도구로 노출할지, 리소스로 노출할지 결정해야 하는 순간이 온다. 판단 기준은 명확하다. 시스템 상태를 변경하는 작업은 도구로, 읽기 전용 데이터 제공은 리소스로, 반복되는 프롬프트 패턴은 프롬프트로 노출한다. 예를 들어, GitHub 이슈를 "조회"하는 기능은 리소스로도 도구로도 구현할 수 있다. 그러나 이슈를 "생성"하는 기능은 반드시 도구로 구현해야 한다. 시스템 상태를 변경하기 때문이다.

### 4.6 생태계 현황

MCP 생태계는 2024년 11월 공개 이후 폭발적으로 성장하였다. 2026년 현재의 주요 지표는 다음과 같다.

- **커뮤니티 서버**: 17,000개 이상의 MCP 서버가 공개되어 있다
- **SDK 다운로드**: MCP SDK는 월간 9,700만 회 이상 다운로드되고 있다
- **호스트 지원**: VS Code, Cursor, Claude Desktop, Windsurf, Zed, JetBrains IDE 등 주요 AI 코딩 도구가 모두 MCP를 지원한다
- **플랫폼 채택**: OpenAI, Google, Microsoft, Anthropic 등 주요 AI 플랫폼이 MCP를 채택하였다

이 숫자가 의미하는 바는 중요하다. 하나의 MCP 서버를 만들면 어떤 AI 도구에서든 사용할 수 있다는 것이다. Copilot용 서버, Cursor용 서버를 따로 만들 필요가 없다. 이것이 표준 프로토콜의 핵심적인 가치이다.

#### MCP Registry

MCP Registry는 공개된 MCP 서버를 검색하고 색인하는 공식 저장소이다. 각 서버의 기능 설명, 설치 방법, 설정 예시가 포함되어 있어 필요한 서버를 빠르게 찾을 수 있다. VS Code의 확장 프로그램 뷰에서 `@mcp`를 검색하면 GUI 기반으로 서버를 탐색할 수도 있다. 2회차 실습에서 직접 Registry를 탐색해 본다.

---

## 2회차: 실습

### 실습 1: MCP 생태계 탐색

이 실습에서는 MCP Registry와 VS Code의 MCP 서버 탐색 기능을 사용하여, 현재 어떤 MCP 서버가 존재하는지 파악한다.

#### 단계 1: MCP Registry 탐색

웹 브라우저에서 MCP Registry(https://registry.modelcontextprotocol.io/)에 접속한다. 검색창에 "github", "database", "web" 등의 키워드를 입력하여 관련 서버를 탐색한다. 각 서버의 설명, 제공하는 도구 목록, 설정 방법을 확인한다.

다음 세 가지 서버를 찾아서 정보를 정리한다.

1. 웹 자동화 관련 서버 1개
2. 버전 관리(VCS) 관련 서버 1개
3. 데이터 저장/조회 관련 서버 1개

각 서버에 대해 이름, 패키지명, 주요 도구 목록, 전송 방식(STDIO/HTTP)을 기록한다.

#### 단계 2: VS Code 내 MCP 서버 탐색

VS Code의 Command Palette(`Ctrl+Shift+P` / `Cmd+Shift+P`)에서 `MCP: List Servers`를 검색하여 현재 등록된 서버 목록을 확인한다. 아직 등록된 서버가 없다면, 다음 실습에서 등록할 것이므로 빈 목록이 정상이다.

### 실습 2: `.vscode/mcp.json` 기본 설정

이 실습에서는 프로젝트에 `.vscode/mcp.json` 파일을 생성하고, 가장 기본적인 Filesystem MCP 서버를 연결하여 동작을 확인한다.

#### 단계 1: 프로젝트 준비

실습용 프로젝트 디렉터리를 생성하고 VS Code로 연다.

```bash
mkdir ~/mcp-practice && cd ~/mcp-practice
mkdir .vscode
code .
```

#### 단계 2: mcp.json 파일 작성

`.vscode/mcp.json` 파일을 다음 내용으로 생성한다. 이 파일은 Filesystem MCP 서버를 등록하는 최소한의 설정이다.

```json
{
  "servers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "${workspaceFolder}"
      ]
    }
  }
}
```

이 설정의 각 요소를 분석한다.

- `"servers"`: MCP 서버 목록을 담는 최상위 객체이다
- `"filesystem"`: 서버의 식별 이름이다. 사용자가 자유롭게 정할 수 있다
- `"command"`: 서버를 실행하는 명령어이다. `npx`는 Node.js 패키지를 임시로 다운로드하여 실행한다
- `"args"`: 명령어에 전달하는 인자 배열이다. `-y`는 설치 확인을 자동 승인하고, `@modelcontextprotocol/server-filesystem`은 패키지 이름이며, `${workspaceFolder}`는 VS Code가 현재 작업 디렉터리 경로로 치환하는 변수이다

#### 단계 3: Agent Mode 활성화 확인

VS Code 설정에서 `chat.agent.enabled` 항목이 `true`인지 확인한다. 설정 방법은 두 가지이다.

1. VS Code 설정 UI에서 `chat.agent.enabled`를 검색하여 체크박스를 활성화한다
2. `settings.json`에 직접 추가한다: `"chat.agent.enabled": true`

#### 단계 4: 서버 상태 확인

`mcp.json`을 저장하면 VS Code 하단 상태 바에 MCP 서버 아이콘이 나타난다. 이 아이콘을 클릭하면 Filesystem 서버의 실행 상태를 확인할 수 있다. 녹색 표시가 나타나면 서버가 정상적으로 실행된 것이다. 만약 오류가 발생하면 출력 패널(Output Panel)에서 MCP 관련 로그를 확인한다.

#### 단계 5: 도구 호출 테스트

Copilot Chat을 Agent Mode로 전환하고 다음과 같이 요청한다.

> "현재 프로젝트의 파일 목록을 보여줘"

Copilot은 Filesystem MCP 서버의 `list_directory` 도구를 호출하여 프로젝트 디렉터리의 파일 목록을 반환한다. 도구 실행 전에 승인 대화상자가 나타나면 "Continue"를 클릭한다.

이어서 다음 요청도 시도한다.

> "README.md 파일을 만들어서 이 프로젝트가 MCP 학습용이라고 적어 줘"

Copilot이 Filesystem MCP의 파일 쓰기 도구와 기본 파일 편집 기능을 어떻게 조합하는지 관찰한다.

#### 단계 6: 비밀 정보 처리 방식 이해

다음 주(5주차)에 GitHub MCP 서버를 연결할 때 API 토큰이 필요하다. 토큰을 코드에 직접 작성하면 보안상 위험하므로, `mcp.json`의 `inputs` 배열을 사용한다. 미리 구조를 확인하여 둔다.

```json
{
  "inputs": [
    {
      "type": "promptString",
      "id": "github-token",
      "description": "GitHub Personal Access Token",
      "password": true
    }
  ],
  "servers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "${workspaceFolder}"]
    }
  }
}
```

`"password": true`를 지정하면 입력 시 마스킹 처리가 된다. `${input:github-token}` 형식으로 서버 설정에서 참조하면, 서버 실행 시 사용자에게 안전하게 값을 입력받는다.

#### 실패 사례: 흔한 설정 오류

MCP 서버 연결에서 가장 빈번하게 발생하는 오류와 해결 방법을 정리한다.

**표 4.4** mcp.json 설정 시 흔한 오류

| 증상 | 원인 | 해결 |
|------|------|------|
| 서버가 시작되지 않음 | Node.js 미설치 또는 18 미만 | `node --version`으로 확인, 18+ 설치 |
| "command not found: npx" | Node.js PATH 미등록 | 터미널에서 `which npx` 확인, PATH 설정 |
| 서버 시작 후 즉시 종료 | 패키지 다운로드 실패 (네트워크) | 네트워크 확인, `npx` 캐시 정리 |
| 도구 목록이 비어 있음 | Agent Mode 미활성화 | `chat.agent.enabled: true` 확인 |
| JSON 파싱 오류 | mcp.json 문법 오류 (쉼표, 따옴표) | VS Code JSON 검증 기능으로 확인 |

---

## 과제

**MCP 아키텍처 다이어그램 + mcp.json 설정 파일 제출**

1. MCP의 호스트-클라이언트-서버 3계층 아키텍처를 다이어그램으로 그린다. 다이어그램에는 최소 3개의 MCP 서버를 포함하고, 각 서버가 제공하는 기능 유형(Tool/Resource/Prompt)을 명시한다. 그리기 도구는 자유이며, draw.io, Mermaid, 손 그림 스캔 모두 허용한다.

2. 실습에서 작성한 `.vscode/mcp.json` 파일을 제출한다. Filesystem 서버가 정상 등록되어 있어야 한다.

3. MCP Registry에서 탐색한 서버 3개의 정보를 정리한 표를 제출한다 (서버명, 패키지명, 주요 도구, 전송 방식).

**제출**: GitHub 리포지토리에 커밋

---

## 핵심 정리

- MCP(Model Context Protocol)는 AI 에이전트와 외부 도구를 연결하는 범용 표준 프로토콜로, N x M 연동 문제를 N + M으로 해결한다
- 호스트(사용자 앱) → 클라이언트(통신 담당) → 서버(기능 제공)의 3계층 구조로 동작한다
- 전송 방식은 STDIO(로컬), SSE(원격 레거시), Streamable HTTP(원격 표준) 세 가지이다
- 프리미티브는 Tool(동작), Resource(데이터), Prompt(템플릿)로 구분되며, 각각 다른 용도와 승인 정책을 가진다
- MCP는 Anthropic이 2024년에 공개하였고, 2025년 AAIF로 거버넌스가 이관되어 개방형 산업 표준으로 자리잡았다
- `.vscode/mcp.json` 파일에 서버를 등록하면 Agent Mode에서 자동으로 도구가 사용 가능해진다

---

## 참고 자료

- Anthropic. (2024). Introducing the Model Context Protocol. *Anthropic Blog*. https://www.anthropic.com/news/model-context-protocol
- Model Context Protocol. (2025). MCP Specification (2025-11-25). *GitHub*. https://github.com/modelcontextprotocol/specification
- Linux Foundation. (2025). Announcing the Agentic AI Foundation. *Linux Foundation Blog*. https://www.linuxfoundation.org/press/announcing-agentic-ai-foundation
- MCP Registry. https://registry.modelcontextprotocol.io/
- VS Code MCP 문서. https://code.visualstudio.com/docs/copilot/chat/mcp-servers

---

## 다음 주 예고

이번 주에는 MCP의 개념과 아키텍처를 이해하고, 기본적인 Filesystem MCP 서버를 연결해 보았다. 5주차에서는 실무에서 자주 사용되는 MCP 서버 세 가지(Playwright, GitHub, Memory)를 직접 연결하고 활용한다. 서버 선택 기준, 보안 고려사항, 다중 서버 설정 방법도 함께 학습한다.
