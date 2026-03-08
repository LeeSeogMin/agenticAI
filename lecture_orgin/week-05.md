# 5주차. MCP 서버 연결과 활용

> **1회차** (강의 90분): 주요 MCP 서버 유형 분석, 서버 선택 기준, 보안 고려사항, 다중 서버 설정
> **2회차** (실습 90분): Playwright MCP 웹 자동화, GitHub MCP 이슈 관리, Memory MCP 대화 기억

---

## 학습목표

1. 용도별 MCP 서버의 유형(웹 자동화, VCS, 메모리, DB)을 분류하고 각 유형의 특성을 설명할 수 있다
2. MCP 서버 선택 시 고려해야 하는 기준(신뢰성, 보안, 유지보수)을 적용할 수 있다
3. 여러 MCP 서버를 동시에 등록하고 Agent Mode에서 조합하여 활용할 수 있다
4. Playwright, GitHub, Memory MCP 서버를 `.vscode/mcp.json`에 등록하고 도구를 호출할 수 있다

## 선수지식

- MCP의 호스트-클라이언트-서버 3계층 아키텍처 (4주차 내용)
- Tool, Resource, Prompt 프리미티브의 차이 (4주차 내용)
- `.vscode/mcp.json` 파일의 기본 구조 (4주차 실습)
- GitHub 계정과 Personal Access Token 발급 방법

---

## 1회차: 강의

### 5.1 MCP 서버 유형별 분석

4주차에서 MCP의 개념과 아키텍처를 학습하였다. 이번 주에는 실제로 사용할 수 있는 MCP 서버를 유형별로 분석하고, 프로젝트에 적합한 서버를 선택하여 연결하는 방법을 학습한다.

2026년 현재 17,000개 이상의 MCP 서버가 공개되어 있으나, 실무에서 자주 사용되는 서버는 몇 가지 유형으로 분류할 수 있다.

#### 웹 자동화 서버

웹 브라우저를 조작하여 정보를 수집하거나 웹 기반 작업을 자동화하는 서버이다. 대표적으로 Playwright MCP가 있다. Playwright MCP는 Microsoft가 관리하는 공식 서버로, Chromium 기반 브라우저를 프로그래밍 방식으로 제어한다. 스크린샷 방식이 아닌 접근성 트리(accessibility tree) 기반으로 페이지를 분석하므로, 이미지 처리 비용 없이 빠르게 페이지 구조를 파악할 수 있다는 장점이 있다.

웹 자동화 서버가 유용한 시나리오는 다음과 같다. 경쟁사 웹사이트의 가격 정보를 수집해야 할 때, 특정 웹 서비스에 로그인하여 데이터를 다운로드해야 할 때, 웹 애플리케이션의 E2E(End-to-End) 테스트를 수행해야 할 때가 그 예이다.

#### 버전 관리(VCS) 서버

GitHub, GitLab 등 버전 관리 플랫폼의 API를 AI 에이전트에 노출하는 서버이다. GitHub MCP 서버를 연결하면 Copilot이 이슈 조회, 이슈 생성, PR 관리, 커밋 이력 분석 등을 수행할 수 있다. 코드 작성 능력에 프로젝트 관리 능력이 더해지는 셈이다.

VCS 서버의 핵심 가치는 "맥락 전환(context switching)의 제거"에 있다. 기존에는 코드를 작성하다가 이슈를 확인하려면 브라우저에서 GitHub을 열어야 했다. VCS MCP 서버가 있으면 Copilot 대화창에서 바로 "열린 이슈 중 버그 라벨이 붙은 것을 보여줘"라고 요청하면 된다.

#### 메모리/저장 서버

AI 에이전트에 영속 메모리(persistent memory)를 제공하는 서버이다. 일반적으로 Copilot Chat은 새 세션을 시작하면 이전 대화 내용을 기억하지 못한다. Memory MCP 서버는 지식 그래프(Knowledge Graph) 형태로 정보를 로컬 파일 시스템에 저장하여 이 한계를 극복한다.

메모리 서버가 특히 유용한 상황은 장기 프로젝트이다. 프로젝트의 기술 스택, 코딩 컨벤션, 아키텍처 결정 사항, 팀 규칙 등을 한번 저장해 두면 매번 반복 설명할 필요가 없어진다. 새로운 대화 세션에서도 이전에 저장한 맥락을 자동으로 참조할 수 있다.

#### 데이터베이스 서버

PostgreSQL, MySQL, SQLite 등 데이터베이스에 접속하여 쿼리를 실행하고 결과를 반환하는 서버이다. 데이터 분석 작업에서 특히 유용하며, AI에이전트가 "지난달 매출 상위 10개 상품을 조회해 줘"라는 자연어 요청을 SQL 쿼리로 변환하여 실행할 수 있게 된다.

#### 협업 도구 서버

Notion, Slack, Linear, Jira 등 협업 도구와 연동하는 서버이다. 프로젝트 문서를 자동으로 작성하거나, 알림을 전송하거나, 작업 상태를 업데이트하는 등의 기능을 제공한다.

**표 5.1** 유형별 대표 MCP 서버

| 유형 | 서버 | 패키지 | 주요 도구 |
|------|------|--------|----------|
| 웹 자동화 | Playwright | `@playwright/mcp` | navigate, snapshot, click, type |
| VCS | GitHub | `@modelcontextprotocol/server-github` | list_issues, create_issue, list_commits |
| 메모리 | Memory | `@modelcontextprotocol/server-memory` | create_entities, read_graph, search_nodes |
| 파일 시스템 | Filesystem | `@modelcontextprotocol/server-filesystem` | read_file, write_file, list_directory |
| 협업 | Notion | `@notionhq/notion-mcp-server` | search, create_page, update_page |
| 디자인 | Figma | `@anthropic/figma-mcp` | get_file, get_components |
| 모니터링 | Sentry | `@sentry/mcp-server-sentry` | list_issues, get_event |

### 5.2 MCP 서버 선택 기준

17,000개 이상의 서버 중에서 프로젝트에 적합한 서버를 선택하는 것은 쉽지 않다. 동일한 기능을 제공하는 서버가 여러 개 존재할 수 있으며, 품질과 신뢰성이 천차만별이다. 다음 기준을 적용하면 합리적인 선택을 할 수 있다.

#### 공식 서버 우선

해당 서비스의 공식 조직이 만든 MCP 서버를 우선 선택한다. Playwright MCP는 Microsoft/Playwright 팀이, GitHub MCP는 MCP 공식 팀이, Notion MCP는 Notion 팀이 관리한다. 공식 서버는 API 변경 시 빠르게 업데이트되고, 보안 취약점이 발견되면 신속하게 패치가 이루어진다.

#### 업데이트 빈도

GitHub 저장소에서 최근 커밋 날짜와 릴리스 주기를 확인한다. 6개월 이상 업데이트가 없는 서버는 방치된 것일 수 있다. MCP 스펙이 빠르게 발전하고 있으므로, 최신 스펙(2025-11-25)을 지원하는지 확인하는 것이 중요하다.

#### 커뮤니티 활성도

GitHub의 스타 수, 이슈 응답 속도, 다운로드 수를 확인한다. 활발한 커뮤니티가 있는 서버는 문제 발생 시 해결 방법을 쉽게 찾을 수 있다.

#### 도구 설명의 품질

MCP 서버의 도구 설명(description)은 AI 모델이 읽고 판단하는 문서이다. 도구 설명이 모호하거나 불완전하면 AI가 잘못된 도구를 선택하거나 잘못된 인자를 전달할 수 있다. MCP Inspector(6주차에서 다룸)로 도구 설명을 미리 확인하는 것을 권장한다.

**표 5.2** MCP 서버 선택 체크리스트

| 기준 | 확인 항목 | 권장 |
|------|----------|------|
| 출처 | 공식 조직 제작 여부 | 공식 서버 우선 |
| 갱신 | 최근 커밋 날짜 | 3개월 이내 |
| 스펙 | MCP 스펙 버전 | 2025-11-25 이상 |
| 커뮤니티 | GitHub Stars / 이슈 응답 | 활성 커뮤니티 |
| 도구 설명 | description 품질 | Inspector로 확인 |
| 의존성 | 외부 의존성 수 | 최소화 |

### 5.3 보안 고려사항

MCP 서버는 AI 에이전트에 강력한 외부 접근 권한을 부여한다. 이는 생산성을 크게 높여 주지만, 동시에 보안 위험도 수반한다. MCP 서버를 연결할 때 반드시 고려해야 하는 보안 사항을 정리한다.

#### API 키 관리

가장 기본적이면서도 가장 중요한 사항이다. API 키, 토큰, 비밀번호를 절대 코드에 직접 작성하지 않는다. `mcp.json`의 `inputs` 배열을 사용하여 실행 시점에 안전하게 입력받거나, 환경 변수로 전달한다. `.vscode/mcp.json` 파일을 Git에 커밋할 때 비밀 정보가 포함되어 있지 않은지 반드시 확인한다.

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
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${input:github-token}"
      }
    }
  }
}
```

#### 최소 권한 원칙

MCP 서버에 전달하는 토큰의 권한은 필요한 최소한으로 제한한다. GitHub PAT를 발급할 때 모든 권한을 부여하는 것이 아니라, 실습에 필요한 이슈 읽기/쓰기와 메타데이터 읽기 권한만 부여한다. Fine-grained token을 사용하면 특정 저장소에 대해서만 권한을 부여할 수도 있다.

#### 도구 실행 승인

MCP의 설계 원칙상, 도구 실행 전에 사용자의 명시적 승인이 필요하다. 세션 내 자동 승인(`Always allow`)을 설정할 수 있지만, 처음 사용하는 서버에 대해서는 개별 승인을 유지하는 것을 권장한다. 어떤 도구가 어떤 인자로 호출되는지 확인하는 습관을 들이는 것이 중요하다.

#### 커뮤니티 서버의 위험성

공식 서버가 아닌 커뮤니티 서버를 사용할 때는 추가적인 주의가 필요하다. 악의적인 MCP 서버가 사용자의 파일을 읽거나, 네트워크에 데이터를 전송하거나, 시스템에 악성 코드를 설치할 가능성이 이론적으로 존재한다. 알려지지 않은 서버를 사용하기 전에 GitHub 저장소의 소스 코드를 검토하거나, 신뢰할 수 있는 출처의 서버만 사용한다.

### 5.4 다중 서버 설정과 상호작용

실무에서는 하나의 프로젝트에 여러 MCP 서버를 동시에 등록하여 사용하는 것이 일반적이다. 웹 자동화, 이슈 관리, 메모리 저장을 하나의 워크플로우에서 조합하여 사용할 수 있다.

#### 다중 서버 등록

`mcp.json`의 `servers` 객체에 여러 서버를 나열하면 된다. 각 서버는 독립적으로 실행되며, 호스트 내부에서 각각의 MCP 클라이언트가 1:1로 연결된다.

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
    "playwright": {
      "command": "npx",
      "args": ["-y", "@playwright/mcp@latest"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${input:github-token}"
      }
    },
    "memory": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory"]
    }
  }
}
```

**그림 5.1** 다중 MCP 서버 등록 예시

#### 서버 간 조합 활용

여러 MCP 서버가 등록되어 있으면, AI 모델은 사용자의 요청에 따라 여러 서버의 도구를 순차적으로 또는 조합하여 사용한다. 예를 들어 다음과 같은 요청이 가능하다.

> "Hacker News에서 상위 5개 기사를 가져와서, 각 기사에 대한 GitHub 이슈를 만들고, 이 작업 내역을 기억해 둬"

이 요청을 처리하기 위해 Copilot은 세 가지 서버의 도구를 순차적으로 호출한다.

1. **Playwright MCP**: `browser_navigate` → `browser_snapshot`으로 Hacker News 접속 및 기사 추출
2. **GitHub MCP**: `create_issue`를 5회 호출하여 각 기사에 대한 이슈 생성
3. **Memory MCP**: `create_entities` → `create_relations`로 작업 내역 저장

이러한 서버 간 조합이 MCP의 진정한 힘이다. 개별 서버는 단순한 기능만 제공하지만, 여러 서버를 조합하면 복잡한 워크플로우를 자연어 한 문장으로 실행할 수 있다.

#### 서버 수 관리

서버를 너무 많이 등록하면 성능과 안정성에 영향을 줄 수 있다. 각 서버는 독립된 프로세스로 실행되므로 메모리를 소비하고, 도구 목록이 길어지면 AI 모델이 적절한 도구를 선택하는 데 어려움을 겪을 수 있다. 프로젝트에 실제로 필요한 서버만 등록하고, 사용하지 않는 서버는 제거하거나 비활성화하는 것이 좋다. 일반적으로 3~5개 이내로 유지하는 것을 권장한다.

---

## 2회차: 실습

### 실습 1: Playwright MCP로 웹 자동화

이 실습에서는 Playwright MCP 서버를 연결하여 Copilot이 웹 브라우저를 직접 조작하도록 한다.

#### 단계 1: mcp.json에 Playwright 서버 등록

4주차 실습에서 만든 프로젝트의 `.vscode/mcp.json`에 Playwright 서버를 추가한다.

```json
{
  "servers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "${workspaceFolder}"]
    },
    "playwright": {
      "command": "npx",
      "args": ["-y", "@playwright/mcp@latest"]
    }
  }
}
```

`@playwright/mcp@latest`는 항상 최신 버전의 Playwright MCP 서버를 다운로드하여 실행한다. Node.js 18 이상이 설치되어 있으면 별도의 설치 과정 없이 `npx`가 패키지를 임시로 가져와 실행하므로 곧바로 사용할 수 있다.

#### 단계 2: 웹 페이지 탐색

Agent Mode에서 다음과 같이 요청한다.

> "https://news.ycombinator.com 에 접속해서 현재 상위 5개 뉴스 제목을 가져와 줘"

Copilot은 Playwright MCP 서버가 제공하는 도구들을 순차적으로 호출한다. 먼저 `browser_navigate` 도구로 해당 URL에 접속하고, `browser_snapshot` 도구로 페이지의 접근성 트리를 읽어 들인다. 이 과정에서 Copilot은 사용자에게 도구 실행 승인을 요청한다. "Continue"를 클릭하여 승인한다.

Copilot이 페이지 내용을 분석하면 상위 5개 뉴스 제목을 추출하여 보여준다.

#### 단계 3: 결과를 파일로 저장

이어서 다음과 같이 후속 작업을 요청한다.

> "이 결과를 news_report.md 파일로 정리해 줘. 각 기사의 제목, URL, 포인트 수를 표로 정리해."

Copilot은 추출한 데이터를 마크다운 형식으로 구성하여 파일을 생성한다. 이 과정에서 MCP 도구(웹 탐색)와 기본 파일 편집 기능이 자연스럽게 결합되는 것을 관찰할 수 있다.

#### 단계 4: 호출된 도구 분석

Copilot Chat 패널에서 각 단계에서 호출된 도구의 이름과 인자를 확인한다. 일반적으로 다음과 같은 도구가 사용된다.

**표 5.3** Playwright MCP 서버의 주요 도구

| 도구 이름 | 기능 | 호출 시점 |
|----------|------|----------|
| `browser_navigate` | 지정한 URL로 이동 | 웹 페이지 접속 시 |
| `browser_snapshot` | 현재 페이지의 접근성 트리 반환 | 페이지 내용 파악 시 |
| `browser_click` | 특정 요소 클릭 | 링크/버튼 상호작용 시 |
| `browser_type` | 텍스트 입력 | 검색어/폼 입력 시 |
| `browser_go_back` | 이전 페이지로 이동 | 탐색 중 되돌아갈 때 |

접근성 트리 기반 분석의 의미를 이해한다. Playwright MCP는 페이지의 스크린샷을 찍어 이미지를 분석하는 것이 아니라, 브라우저가 구축한 접근성 트리(DOM의 시맨틱 구조)를 텍스트로 변환하여 AI 모델에 전달한다. 이 방식은 이미지 처리 비용이 들지 않아 빠르고, 토큰 소비도 적으며, 텍스트 기반이므로 구조적 분석에 유리하다.

### 실습 2: GitHub MCP로 이슈 관리

이 실습에서는 GitHub MCP 서버를 연결하여 Copilot이 저장소의 이슈와 커밋을 관리하도록 한다.

#### 단계 1: Personal Access Token 발급

GitHub MCP 서버는 GitHub API에 접근하기 위해 PAT(Personal Access Token)가 필요하다. 다음 절차로 토큰을 발급받는다.

1. GitHub에 로그인한다
2. Settings > Developer settings > Personal access tokens > Fine-grained tokens으로 이동한다
3. "Generate new token"을 클릭한다
4. Token name: `mcp-practice` (또는 자유 이름)
5. Repository access: 실습용 저장소만 선택한다 (최소 권한 원칙)
6. Permissions에서 다음을 설정한다:
   - Issues: Read and write
   - Metadata: Read-only
   - Pull requests: Read-only (선택)
7. "Generate token"을 클릭하고, 발급된 토큰을 안전한 곳에 복사해 둔다

**주의**: 토큰은 발급 직후에만 전체 값을 볼 수 있다. 페이지를 벗어나면 다시 확인할 수 없으므로 반드시 즉시 복사한다.

#### 단계 2: mcp.json에 GitHub 서버 등록

`.vscode/mcp.json`에 GitHub 서버를 추가한다.

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
    },
    "playwright": {
      "command": "npx",
      "args": ["-y", "@playwright/mcp@latest"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${input:github-token}"
      }
    }
  }
}
```

파일을 저장하면 VS Code가 GitHub PAT를 입력받는 대화상자를 표시한다. 앞서 발급받은 토큰을 입력한다. `"password": true` 덕분에 입력 시 마스킹 처리가 된다.

#### 단계 3: 이슈 조회

Agent Mode에서 다음과 같이 요청한다.

> "이 저장소의 열린 이슈 목록을 보여줘"

Copilot은 GitHub MCP 서버의 `list_issues` 도구를 호출하여 현재 열려 있는 이슈 목록을 가져온다. 각 이슈의 번호, 제목, 라벨, 담당자 정보가 표시된다. 아직 이슈가 없다면 빈 목록이 반환되는 것이 정상이다.

#### 단계 4: 이슈 생성

새로운 이슈를 생성한다.

> "새 이슈를 만들어 줘: 'Playwright MCP 실습 완료' 제목으로, 라벨은 enhancement, 본문에는 실습 날짜와 테스트한 도구 목록을 포함해 줘"

Copilot은 `create_issue` 도구를 호출하여 지정한 내용으로 이슈를 생성한다. 도구 호출 전 승인 대화상자에서 어떤 인자가 전달되는지 확인한다. 의도하지 않은 내용이 포함되어 있지 않은지 검토한 후 승인한다.

#### 단계 5: 커밋 요약

최근 커밋 내역을 요약한다.

> "최근 커밋 5개를 요약해 줘"

Copilot은 `list_commits` 도구로 최근 5개 커밋을 조회하고, 각 커밋의 메시지와 변경 내용을 요약하여 보여준다. 이처럼 GitHub MCP 서버를 연결하면 Copilot이 코드 작성뿐 아니라 프로젝트 관리 작업까지 수행할 수 있게 된다.

### 실습 3: Memory MCP로 대화 기억

이 실습에서는 Memory MCP 서버를 연결하여 Copilot이 대화 세션 간에 정보를 기억하도록 한다.

#### 단계 1: mcp.json에 Memory 서버 등록

`.vscode/mcp.json`에 Memory 서버를 추가한다. 이전 단계에서 Playwright와 GitHub을 이미 등록했으므로, `servers` 객체에 하나만 추가하면 된다.

```json
"memory": {
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-memory"]
}
```

Memory MCP 서버는 별도의 API 키나 인증이 필요 없다. 지식 그래프를 로컬 파일 시스템에 JSON 형태로 저장하므로, 서버 실행만 하면 곧바로 사용할 수 있다.

#### 단계 2: 정보 저장

Agent Mode에서 다음과 같이 프로젝트 정보를 알려준다.

> "이 프로젝트는 MCP 학습용이야. 사용하는 서버는 Playwright, GitHub, Memory 세 가지이고, Node.js 18과 VS Code Insiders를 사용한다. 이 정보를 기억해 둬."

Copilot은 Memory MCP 서버의 `create_entities` 도구를 호출하여 "프로젝트", "Playwright", "GitHub", "Memory", "Node.js", "VS Code" 등의 개체(entity)를 생성한다. 이어서 `create_relations` 도구로 이들 간의 관계("프로젝트 → 사용 → Playwright" 등)를 설정한다.

#### 단계 3: 새 세션에서 정보 조회

Copilot Chat에서 새로운 세션을 시작한다. 이전 대화 이력이 없는 상태에서 다음과 같이 질문한다.

> "이 프로젝트에서 사용하는 MCP 서버가 뭐였지?"

Copilot은 Memory MCP 서버의 `read_graph` 또는 `search_nodes` 도구를 호출하여 이전에 저장한 지식 그래프에서 관련 정보를 검색한다. 세션이 바뀌었음에도 불구하고 "Playwright, GitHub, Memory" 서버 정보가 정확하게 반환되는 것을 확인한다.

#### 단계 4: 정보 업데이트

기존 정보에 새로운 내용을 추가한다.

> "이 프로젝트에 Notion MCP 서버도 추가할 계획이야. 기억해 둬."

Memory MCP 서버는 기존 지식 그래프에 새로운 개체와 관계를 추가한다. 지식 그래프는 누적 방식으로 동작하므로, 기존 정보가 덮어씌워지지 않고 새 정보가 추가된다.

#### 실패 사례: Memory 서버의 한계

Memory MCP 서버는 유용하지만 한계도 있다. 저장된 정보가 많아지면 검색 결과의 정확도가 떨어질 수 있다. 또한 지식 그래프의 구조가 사용자의 의도와 다르게 형성될 수 있다. 예를 들어, "Python"이라는 단어를 프로그래밍 언어로 저장할 의도였지만, 모델이 "뱀"으로 해석하여 잘못된 관계를 생성할 수 있다. 이런 경우 `delete_entities` 도구로 잘못된 정보를 삭제하고 다시 저장해야 한다.

### 종합 실습: 세 서버 조합 활용

마지막으로 세 가지 MCP 서버를 조합하여 하나의 워크플로우를 수행한다.

> "GitHub에서 이 저장소의 열린 이슈 중 가장 오래된 것을 찾아서, 관련 웹 페이지가 있으면 내용을 확인하고, 이 조사 결과를 기억해 둬"

이 요청은 GitHub MCP(이슈 조회) → Playwright MCP(웹 페이지 확인) → Memory MCP(결과 저장)의 순서로 세 서버의 도구가 순차적으로 호출되는 것을 관찰한다. 각 단계에서 어떤 도구가 어떤 인자로 호출되는지 Copilot Chat 패널에서 확인한다.

---

## 과제

**MCP 서버 3종 연결 + 각 도구 호출 결과 제출**

1. `.vscode/mcp.json`에 Playwright, GitHub, Memory 세 가지 서버를 모두 등록한 최종 설정 파일을 제출한다.

2. 각 서버에서 최소 하나의 도구를 호출한 결과를 스크린샷으로 제출한다.
   - Playwright: 웹 페이지 탐색 결과
   - GitHub: 이슈 조회 또는 생성 결과
   - Memory: 정보 저장 및 조회 결과

3. (도전 과제) 세 서버를 조합하여 자연어 한 문장으로 복합 작업을 수행한 결과를 제출한다. 어떤 도구가 어떤 순서로 호출되었는지 정리하여 함께 제출한다.

**제출**: GitHub 리포지토리에 커밋

---

## 핵심 정리

- MCP 서버는 웹 자동화, VCS, 메모리, DB, 협업 도구 등 유형별로 분류할 수 있다
- 서버 선택 시 공식 서버 우선, 업데이트 빈도, 커뮤니티 활성도, 도구 설명 품질을 확인한다
- API 키는 `mcp.json`의 `inputs` 배열과 `${input:id}` 변수로 안전하게 처리한다
- 최소 권한 원칙을 적용하여 토큰 권한을 필요한 범위로 제한한다
- 여러 MCP 서버를 동시에 등록하면 AI가 서버 간 도구를 조합하여 복합 작업을 수행할 수 있다
- Playwright MCP는 접근성 트리 기반으로 웹 페이지를 분석하여 빠르고 토큰 효율적이다
- Memory MCP는 지식 그래프 형태로 정보를 영속 저장하여 세션 간 맥락을 유지한다

---

## 참고 자료

- Playwright MCP. https://github.com/microsoft/playwright-mcp
- GitHub MCP Server. https://github.com/github/github-mcp-server
- Memory MCP Server. https://github.com/modelcontextprotocol/servers/tree/main/src/memory
- MCP Registry. https://registry.modelcontextprotocol.io/
- GitHub Fine-grained PAT 문서. https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens

---

## 다음 주 예고

이번 주에는 이미 만들어진 MCP 서버를 연결하여 Copilot의 기능을 확장하는 방법을 배웠다. 6주차에서는 한 단계 더 나아가, Python MCP SDK를 사용하여 나만의 MCP 서버를 직접 만들어 본다. 외부 API를 MCP 서버로 래핑하는 과정을 통해 "도구를 사용하는 사람"에서 "도구를 만드는 사람"으로 전환하는 경험을 하게 된다.
