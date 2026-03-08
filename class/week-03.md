# Week 3. MCP, Skills, Plugins 실전 입문

> 원본: docs/ch3.md

## 학습 목표

- MCP가 무엇을 해결하는지 실제 사례로 설명할 수 있다
- GitHub Copilot에서 기존 MCP 서버를 연결하고 실제로 호출할 수 있다
- GitHub Copilot용 skill 또는 instruction 파일을 작성하고 적용 효과를 비교할 수 있다
- plugin/app/connector, instructions, hooks, memory 계열이 MCP 및 Skills와 어떻게 다른지 구분할 수 있다
- 최소 MCP 서버를 직접 구현하고 테스트할 수 있다

---

## 선수 지식

- 2주차의 에이전트 스타터 프로젝트
- Python 기초 문법
- 가상환경과 `.env` 사용법

---

## 3.1 왜 이제는 MCP만 알아서는 부족한가

### 3.1.1 도구만 연결해도 일이 잘 안 되는 이유

- 에이전트에게 도구를 연결했다고 해서 자동으로 좋은 결과가 나오지는 않음
- 이유:
  - 어떤 도구를 언제 써야 하는지 판단이 불명확할 수 있음
  - 출력 형식이 들쭉날쭉할 수 있음
  - 검증 절차 없이 결과를 바로 수락할 수 있음
- 즉, 도구 연결만으로는 "할 수 있는 일"이 늘어날 뿐이고, "잘 일하는 방식"까지 보장하지는 않음

### 3.1.2 규칙 없는 에이전트의 한계

- 규칙이 없으면 에이전트는 같은 요청에도 매번 다른 방식으로 행동할 수 있음
- 흔한 문제:
  - 출력 위치가 바뀜
  - 파일 이름이 일관되지 않음
  - 검증 없이 바로 종료함
  - 위험한 동작을 무심코 시도함
- 그래서 **도구 연결(MCP)**과 함께 **작업 규칙(Skills / Instructions)**이 필요함

### 3.1.3 제품마다 확장 방식이 다른 현실

- 2026년의 도구 생태계는 아직 완전히 하나로 통일되어 있지 않음
- 같은 기능이라도 제품마다 붙이는 방식이 다를 수 있음
  - 어떤 제품은 MCP 서버를 직접 붙임
  - 어떤 제품은 skill 파일이나 instruction 파일을 함께 사용함
  - 어떤 제품은 hooks나 settings로 기본 동작을 고정함
  - 어떤 제품은 memory나 spaces로 지속 문맥을 유지함
  - 어떤 제품은 plugin, app, connector 같은 이름으로 기능을 포장함
- 따라서 실무 역량은 "한 제품의 메뉴를 외우는 것"이 아니라, 각 층위의 역할을 구분하는 데서 시작함

---

## 3.2 MCP, Skills, Plugins, Instructions, Hooks, Memory의 역할 구분

### 3.2.1 MCP

- MCP(Model Context Protocol)는 에이전트가 외부 도구와 데이터를 사용할 수 있게 하는 표준 인터페이스
- 질문으로 바꾸면:
  - "에이전트가 무엇을 호출할 수 있는가?"
- 대표 사례:
  - 파일 읽기
  - GitHub 이슈 조회
  - 데이터베이스 질의
  - 외부 API 호출

### 3.2.2 Skills / Instructions

- Skills 또는 Instructions는 에이전트가 작업할 때 따라야 할 규칙과 절차를 담음
- 질문으로 바꾸면:
  - "에이전트가 어떤 순서와 기준으로 일해야 하는가?"
- 대표 사례:
  - 출력은 반드시 `output/`에 저장
  - 위험한 명령은 사용자 승인 없이 실행하지 않음
  - 테스트를 먼저 실행하고 실패 시 원인을 기록

### 3.2.3 Plugins / Apps / Connectors

- plugin, app, connector는 제품 안에서 기능을 배포하거나 연결하는 표면인 경우가 많음
- 질문으로 바꾸면:
  - "이 기능을 제품 안에서 어떤 형태로 붙이고 배포할 것인가?"
- 이름은 다르지만 대체로 다음 둘 중 하나를 감쌈
  - 도구 연결
  - 사용자용 기능 패키징

### 3.2.4 셋을 혼동하면 생기는 문제

- MCP를 배웠다고 해서 Skills까지 배운 것은 아님
- skill 파일을 만들었다고 해서 외부 도구 연결이 생기는 것도 아님
- plugin을 설치했다고 해서 표준화된 도구 인터페이스를 이해한 것도 아님

### 3.2.5 Instructions / Rules / Settings

- 이 계층은 skill보다 더 넓은 기본 행동 규칙을 담는 경우가 많음
- 예:
  - 항상 한국어로 응답
  - 파괴적 명령은 승인 전 금지
  - 결과를 항상 특정 폴더에 저장

### 3.2.6 Hooks

- hook은 특정 이벤트가 발생했을 때 자동으로 동작하는 연결점
- 예:
  - 명령 실행 직전 확인
  - 파일 수정 뒤 포맷 검사
  - 완료 후 테스트 실행

### 3.2.7 Memory / Spaces / Project Memory

- memory 계층은 프로젝트 문맥을 지속적으로 유지함
- 예:
  - 이 저장소의 출력 위치는 항상 `output/`
  - 기본 테스트 도구는 pytest
  - 위험한 명령은 승인 전 금지

**표 3.1** 주요 개념의 차이

| 개념 | 핵심 질문 | 역할 |
|------|----------|------|
| MCP | 무엇을 호출할 수 있는가 | 도구 연결 |
| Skills / Instructions | 어떻게 일하게 할 것인가 | 작업 규칙 |
| Plugins / Apps / Connectors | 제품에 어떻게 붙일 것인가 | 확장/패키징 |
| Hooks | 언제 자동 검사/실행할 것인가 | 이벤트 연결 |
| Memory / Spaces | 무엇을 계속 기억하게 할 것인가 | 지속 문맥 |

---

## 3.3 MCP 빠른 이해

### 3.3.1 MCP가 해결하는 문제

- AI 도구마다 외부 시스템 연결 방식이 제각각이면 재사용이 어려움
- 같은 기능을 플랫폼마다 다시 구현해야 함
- MCP는 이 문제를 줄이기 위해 도구 연결 방식을 표준화하려는 접근임

### 3.3.2 Tools, Resources, Prompts

- MCP는 보통 세 가지 요소를 중심으로 설명됨
  - **Tools**: AI가 호출하는 함수
  - **Resources**: AI가 읽는 컨텍스트 데이터
  - **Prompts**: 재사용 가능한 프롬프트 자산

- 이 수업에서는 먼저 **Tools** 중심으로 실습함
- 이유:
  - 가장 체감이 빠름
  - 도구 호출 로그를 확인하기 쉬움
  - 이후 LangChain, LangGraph와 연결하기 좋음

### 3.3.3 STDIO와 Remote MCP

- 로컬 실습에서는 보통 STDIO 기반 MCP를 먼저 접함
  - 클라이언트가 서버 프로세스를 실행
  - 표준 입력/출력으로 메시지를 주고받음
- 원격 MCP는 네트워크를 통해 서버에 연결함
  - 인증과 권한 관리가 더 중요해짐

### 3.3.4 승인, 권한, 보안 경계

- MCP를 쓰는 이유 중 하나는 무제한 접근이 아니라 **제한된 접근**을 만들기 위해서임
- 예를 들어:
  - 파일 읽기는 허용
  - 파일 삭제는 금지
  - 특정 폴더 밖 접근은 금지
- 따라서 좋은 MCP 서버는 "많이 할 수 있는 서버"보다 "무엇을 못 하게 할지 분명한 서버"에 가까움

---

## 3.4 실습 1: 기존 MCP 서버 연결해서 써 보기

### 실습 목표

- GitHub Copilot에서 기존 MCP 서버를 실제로 연결하고 호출한다

### 수행 단계

1. 사용할 MCP 서버 하나를 고른다
   - 예: GitHub MCP server, 파일 시스템 계열, 간단한 유틸리티 서버
2. GitHub Copilot 환경에 서버를 등록한다
3. 도구를 2회 이상 호출한다
4. 입력과 출력을 기록한다
5. 실패 사례가 있으면 원인을 메모한다

### GitHub Copilot 최신 실습 방법

- VS Code에서 MCP 서버를 쓰는 최신 공식 흐름은 다음 두 가지 중 하나임
  - MCP Servers Marketplace에서 서버 설치
  - 저장소 또는 설정 파일에 MCP 구성 추가
- 수업에서는 입문 난도를 낮추기 위해 **Marketplace 또는 기본 제공 GitHub MCP server**부터 시작하는 것을 권장함

### GitHub Copilot 실습 예시

1. VS Code에서 확장 패널을 연다
2. MCP Server 필터를 사용해 필요한 서버를 찾는다
3. `github` 서버를 설치하거나 이미 구성된 서버를 확인한다
4. Copilot Chat에서 **Agent mode**를 연다
5. 다음과 같이 요청한다

```text
현재 저장소의 최근 열린 이슈를 확인하고, week3 실습에 도움이 될 만한 항목을 요약해줘.
가능하면 MCP 도구를 사용해줘.
```

6. MCP 도구가 실제로 호출되었는지 확인한다

### 관찰 포인트

- 어떤 설명이 도구 선택에 영향을 주는가
- 승인 요청은 언제 나타나는가
- 결과를 그대로 믿으면 위험한 지점은 어디인가
- 출력이 실제로 유용한가, 아니면 후처리가 더 필요한가

### 결과 기록 예시

```markdown
- 호출 도구: read_file
- 입력: path=docs/notes.md
- 결과: 파일 내용 정상 반환
- 검증: 실제 파일과 내용 비교 완료
- 관찰: 설명이 짧아도 도구 선택은 잘 되었으나, 경로 오입력 시 오류 메시지가 불명확했음
```

---

## 3.5 실습 2: Skill / Instruction 파일 작성

### 실습 목표

- 같은 작업을 "규칙 없이" 했을 때와 "규칙 있게" 했을 때의 차이를 비교한다

### 과제 설명

- 작업 하나를 고른다
  - 예: 파일 요약
  - 예: 테스트 코드 생성
  - 예: 로그 정리
- 먼저 규칙 없이 요청한다
- 다음으로 아래와 같은 규칙 파일을 만든 뒤 같은 요청을 다시 한다

- GitHub Copilot 기준 최신 실습 방식
  - Agent mode에서는 instruction 파일로 먼저 비교 실습을 한다
  - Agent Skills는 공식 문서 기준 Copilot coding agent, Copilot CLI, VS Code Insiders에서 우선 지원되므로, Skills 실습은 **VS Code Insiders 또는 Copilot CLI** 기준으로 수행한다

예시 규칙:

```markdown
# 작업 규칙

- 출력 파일은 반드시 output/ 디렉토리에 저장한다.
- 결과를 바로 확정하지 말고 핵심 검증 항목 3개를 먼저 적는다.
- 불확실한 정보는 추측하지 말고 확인 필요라고 표시한다.
- 실행 후 logs/에 실행 내용을 남긴다.
```

### 비교 항목

- 출력 형식이 더 일관적인가
- 누락이 줄어들었는가
- 검증 항목이 더 명확해졌는가
- 안전성이 높아졌는가

### GitHub Copilot 실습 예시

- 규칙 파일 없이

```text
output/summary.md에 docs/notes.md 요약을 저장해줘.
```

- 규칙 파일 또는 skill 적용 후

```text
docs/agent-rules.md의 규칙을 따르면서 docs/notes.md를 요약해줘.
반드시 output/summary.md에 저장하고, 검증 항목도 함께 적어줘.
```

### Agent Skills 실습 예시

- VS Code Insiders 또는 Copilot CLI를 사용할 수 있다면 다음 구조를 권장함

```text
.github/skills/doc-summary/
  SKILL.md
  examples/
  templates/
```

- `SKILL.md`에는 최소한 다음을 적음
  - 언제 이 skill을 로드할지
  - 어떤 출력 형식을 사용할지
  - 검증 항목을 어떻게 적을지

### 실습 확장: custom instructions 추가

- 같은 규칙을 저장소 수준 instruction으로 옮겨 본다
- 학생은 비교한다
  - skill로 넣었을 때
  - instruction으로 넣었을 때
  - 둘의 역할이 어떻게 다른가

### 실습의 의도

- 이 실습은 "규칙을 적으면 에이전트가 항상 완벽해진다"를 보여주려는 것이 아님
- 오히려 다음 사실을 체감하게 하려는 것임
  - 규칙이 없으면 결과가 흔들린다
  - 규칙을 적으면 검토가 쉬워진다

---

## 3.6 실습 3: 최소 MCP 서버 직접 만들기

### 실습 목표

- 가장 작은 형태의 MCP 서버를 직접 구현하고 테스트한다

### 구현 예시

- 현재 시각 반환
- 지정 디렉토리 파일 목록 반환
- 간단한 계산
- 로컬 메모 읽기

### 설계 원칙

- 이름이 명확해야 함
- 설명이 구체적이어야 함
- 입력이 단순해야 함
- 오류 메시지가 읽기 쉬워야 함
- 가능한 경우 읽기 전용으로 시작하는 것이 안전함

### 최소 서버의 의미

- 이 실습의 목적은 대형 서버를 만드는 것이 아님
- 목적은 "에이전트가 도구를 부르는 구조"를 눈으로 확인하는 것
- 즉, 최소 서버는 개념 증명이자 이후 확장의 출발점임

### 설계 문서에 포함할 항목

- 도구 이름
- 도구 설명
- 입력 파라미터
- 예상 출력
- 금지하거나 제한할 동작

### GitHub Copilot 활용 방식

- Agent mode에서 다음과 같이 요청한다

```text
Python으로 최소 MCP 서버 예제를 만들어줘.
요구사항:
- 읽기 전용 도구 1개만 제공
- 입력과 오류 처리가 명확해야 함
- 테스트 방법도 docs/server-design.md에 적어줘
```

- 학생은 Copilot이 만든 코드를 그대로 수락하지 말고 다음을 직접 확인해야 함
  - 도구 설명이 충분히 구체적인가
  - 입력 검증이 있는가
  - 잘못된 입력에 대한 오류 메시지가 읽기 쉬운가
  - 테스트 절차가 실제로 가능한가

---

## 3.7 실습 4: GitHub Copilot CLI plugin 실습

### 실습 목표

- GitHub Copilot에서 plugin이 무엇을 묶는지 실제로 확인한다
- plugin 안에 skills, MCP 설정, agents를 함께 담을 수 있다는 점을 체험한다

### 왜 CLI plugin을 다루는가

- GitHub Copilot의 plugin은 현재 **Copilot CLI**에서 가장 명확하게 드러남
- 공식 문서 기준 plugin은 재사용 가능한 설치 단위이며, 여기에 다음 요소를 함께 넣을 수 있음
  - agents
  - skills
  - hooks
  - MCP server configurations

### 수행 단계

1. `my-plugin/` 디렉토리를 만든다
2. 루트에 `plugin.json`을 만든다
3. `skills/` 아래에 간단한 skill 하나를 만든다
4. 필요하면 `.mcp.json`에 MCP 서버 설정 하나를 넣는다
5. 로컬에서 plugin을 설치한다
6. plugin이 실제로 로드되었는지 확인한다

### 권장 구조

```text
my-plugin/
  plugin.json
  skills/
    doc-summary/
      SKILL.md
  .mcp.json
```

### 최소 `plugin.json` 예시

```json
{
  "name": "my-dev-tools",
  "description": "Week 3 practice plugin",
  "version": "1.0.0",
  "skills": "skills/",
  "mcpServers": ".mcp.json"
}
```

### 설치와 확인

```bash
copilot plugin install ./my-plugin
copilot plugin list
```

- 대화형 세션 안에서는 다음 명령으로 확인 가능
  - `/plugin list`
  - `/skills list`

### 실습의 핵심 관찰점

- plugin은 단순한 "추가 기능 하나"가 아님
- 실제로는 **skills + MCP + agents를 묶어 배포하는 패키지**에 가깝다
- 따라서 plugin은 MCP나 skill과 경쟁 개념이 아니라 **포장 단위**임

### 왜 이 과제가 필요한가

- 학생은 plugin을 "MCP와 비슷한 것"으로 착각하기 쉬움
- 이 실습을 통해 다음 관계를 분명히 이해해야 함
  - MCP = 도구 연결
  - Skill = 작업 규칙
  - Plugin = 여러 요소를 묶는 설치 단위

---

## 3.8 실습 5: hooks와 memory를 붙여 보기

### 실습 목표

- 규칙을 문서로만 두지 않고 자동 실행과 지속 문맥으로 확장하는 감각을 익힌다

### hooks 실습 아이디어

- 다음 중 하나를 고른다
  - 작업 완료 후 `output/` 생성 여부 검사
  - 결과 파일이 없으면 실패로 간주
  - 테스트가 없으면 경고 출력

### memory 실습 아이디어

- 다음 문맥 중 하나를 에이전트 기본 기억으로 정리한다
  - 이 프로젝트의 출력 위치
  - 기본 검증 절차
  - 금지 명령 또는 주의 명령

### 관찰 포인트

- 문서 규칙만 둘 때와 hook을 붙였을 때 무엇이 달라지는가
- 같은 규칙을 memory에 둘 때 세션 간 일관성이 높아지는가
- skill, instruction, hook, memory가 서로 어떤 역할을 나누는가

---

## 3.9 테스트와 검증

### 3.8.1 AI 없이 단독 테스트하는 법

- MCP 서버는 가능하면 AI 없이도 테스트할 수 있어야 함
- 이유:
  - 문제 원인을 분리하기 쉬움
  - 모델 출력과 서버 오류를 구분할 수 있음

### 3.8.2 정상 입력 / 오류 입력 테스트

- 최소한 두 종류의 테스트가 필요함
  - 정상 입력
  - 잘못된 입력

- 예:
  - 정상 경로 입력 시 파일 목록 반환
  - 존재하지 않는 경로 입력 시 읽기 쉬운 오류 반환

### 3.8.3 로그와 산출물 남기기

- 다음 파일을 남기는 습관을 들임
  - 실행 로그
  - 출력 예시
  - 설계 문서
  - 비교 메모
- 이 기록은 나중에 LangChain, LangGraph 실습으로 넘어갈 때 매우 중요함

---

## 3.10 제출물

- MCP 연결 설정 파일
- skill 또는 instruction 파일 1개
- 최소 MCP 서버 코드
- Copilot CLI plugin 디렉토리 1개
- hook 또는 자동 검증 설계 메모 1개
- project memory 초안 1개
- 테스트 로그
- 비교 결과 문서
- 업데이트된 체크리스트

---

## 3.11 핵심 정리

- MCP는 도구 연결이다
- Skills / Instructions는 작업 규칙이다
- Plugins / Apps / Connectors는 제품별 확장 표면이다
- Hooks는 자동 검사와 자동 실행의 연결점이다
- Memory는 프로젝트 문맥을 지속시키는 계층이다
- 실제 활용 역량은 이 층들을 함께 다룰 때 생긴다
- 3주차의 산출물은 이후 LangChain, LangGraph, 멀티에이전트 실습의 입력 자산이 된다

---

## 부록 A. Claude Code로 수행하는 동일 실습

- 본문 실습은 GitHub Copilot 중심으로 진행하지만, 같은 층위를 Claude Code에서도 거의 그대로 실습할 수 있다
- Claude Code는 터미널 중심 도구이지만 `plugins`, `skills`, `MCP`, `hooks`, `memory`가 비교적 명확하게 드러나므로 구조 학습에 적합하다

### A.1 플러그인 둘러보기

- Claude Code에서 플러그인 UI를 열려면 다음 명령을 사용한다

```text
/plugin
```

- 여기서 설치된 플러그인과 마켓플레이스를 확인할 수 있다
- 공식 Anthropic 플러그인 마켓플레이스는 보통 기본 제공된다

### A.2 플러그인 설치

- 가장 쉬운 방법은 `/plugin` 화면의 `Discover` 탭에서 원하는 플러그인을 찾는 것이다
- 명령으로 바로 설치할 수도 있다

```text
/plugin install <plugin-name>@claude-plugins-official
```

- 설치 후에는 보통 바로 사용할 수 있다
- 학생은 다음을 확인한다
  - 플러그인이 실제로 설치되었는가
  - 새 명령이나 skill이 보이는가

### A.3 플러그인으로 무엇을 묶을 수 있는가

- Claude Code plugin은 보통 다음 요소를 묶어 공유하는 단위다
  - `skills`
  - `agents`
  - `hooks`
  - `MCP servers`
- 따라서 plugin은 "추가 기능 하나"라기보다 **재사용 가능한 작업 패키지**에 가깝다

### A.4 직접 플러그인 만들기

- 기본 구조 예시:

```text
my-first-plugin/
  .claude-plugin/
    plugin.json
  skills/
    hello/
      SKILL.md
```

- 최소 `plugin.json` 예시:

```json
{
  "name": "my-first-plugin",
  "description": "Week 3 plugin practice",
  "version": "1.0.0"
}
```

- 최소 `SKILL.md` 예시:

```markdown
---
description: Summarize a document and save the result to output/
---

Read the target document, summarize it, save the result in output/, and list 3 verification checks.
```

### A.5 로컬에서 플러그인 테스트

- 직접 만든 플러그인은 다음처럼 로컬에서 바로 테스트할 수 있다

```bash
claude --plugin-dir ./my-first-plugin
```

- 실행 후 다음처럼 호출해 본다

```text
/my-first-plugin:hello
```

- 학생은 다음을 확인한다
  - 플러그인 명령이 보이는가
  - skill이 실제로 로드되는가
  - 출력 형식이 의도대로 유지되는가

### A.6 Skills 실습

- Claude Code 공식 문서 기준 skills는 `SKILL.md` 파일로 확장된다
- 기본 실습 구조:

```text
.claude/
  skills/
    doc-summary/
      SKILL.md
```

- 요청 예시:

```text
docs/notes.md를 요약해줘.
가능하면 skill을 활용하고 output/summary.md에 저장해줘.
```

- 필요하면 직접 호출할 수도 있다

```text
/doc-summary
```

### A.7 MCP 서버 연결

- Claude Code에서 MCP 서버를 추가할 때는 다음 흐름을 사용한다

```bash
claude mcp add github --scope local -- <server-command>
claude mcp list
```

- 설정 범위:
  - `local`: 현재 프로젝트 개인 설정
  - `project`: 팀 공유 설정
  - `user`: 여러 프로젝트 공통 설정

- 실습에서는 다음을 확인한다
  - 서버가 실제로 목록에 보이는가
  - 도구 호출이 가능한가
  - 권한 범위가 적절한가

### A.8 Hooks, Settings, Memory

- Claude Code는 plugin, skill, MCP 외에도 다음 계층을 분리해서 볼 수 있다
  - **Hooks**: 특정 이벤트 전후 자동 실행
  - **Settings**: 기본 행동 규칙
  - **Memory**: 프로젝트 맥락 유지

- hooks 실습 예:
  - 작업 완료 후 로그 저장
  - 명령 실행 전 경고 출력
  - 특정 파일 변경 뒤 검사 스크립트 실행

- settings / memory 실습 예:
  - 출력은 항상 `output/`
  - 파괴적 명령은 승인 전 금지
  - 테스트 가능 시 먼저 테스트

### A.9 최소 MCP 서버 실습

- 요청 예시:

```text
Python으로 읽기 전용 최소 MCP 서버를 만들어줘.
도구 설명, 입력 검증, 오류 메시지, 테스트 절차를 함께 포함해줘.
```

- 학생은 다음을 직접 검토한다
  - 설명이 구체적인가
  - 입력 검증이 있는가
  - 오류 메시지가 읽기 쉬운가
  - 테스트 절차가 실제로 가능한가

### A.10 설치 문제 해결

- 문제가 생기면 다음 순서로 점검한다
  1. `/plugin`에서 설치 상태 확인
  2. 플러그인 이름과 마켓플레이스 이름이 정확한지 확인
  3. 로컬 개발 중이면 `claude --plugin-dir ...`로 직접 테스트
  4. MCP가 함께 묶여 있다면 `claude mcp list`로 서버 등록 상태 확인
  5. settings, hooks, skills 경로가 구조와 맞는지 확인

- 무조건 `~/.claude`를 지우는 방식은 마지막 수단으로만 사용한다

### A.11 주의사항

- 영상에서 자주 보이는 다음 표현은 공식 기능으로 단정하면 안 된다
  - `Remote Control`
  - `Ralph Loop`
- 커뮤니티 플러그인, 외부 도구, 개인 워크플로우일 가능성이 있으므로 공식 강의 본문에는 넣지 않는 편이 안전하다

### A.12 Claude Code 부록의 의도

- 같은 과제를 Copilot과 Claude Code에서 각각 수행해 보면 차이가 잘 드러난다
  - Copilot: IDE 중심, Copilot Chat + CLI plugin 쪽이 명확
  - Claude Code: 터미널 중심, MCP·Skills·Plugins·Hooks·Memory가 한 체계 안에서 잘 드러남
- 부록의 목적은 우열 비교가 아니라 **도구별 작업 감각 차이**를 체험하는 것임

---

## 부록 B. ChatGPT / Codex로 수행하는 동일 실습

- OpenAI 쪽은 이름이 조금 다르지만, 같은 층위로 매핑할 수 있다
  - MCP = 외부 도구 연결
  - Skills = Codex app/CLI/IDE에서의 skill 또는 rules 계층
  - Plugin에 가까운 개념 = ChatGPT Apps

### B.1 ChatGPT에서의 "plugin"은 지금 무엇인가

- 2025년 12월부터 OpenAI는 connectors를 **Apps**로 통합해 안내함
- 따라서 ChatGPT 쪽에서 plugin과 가장 비슷한 현재 개념은 **Apps**라고 보는 편이 정확함
- 학생은 여기서 "ChatGPT 안에 외부 기능을 붙인다"는 감각을 익히면 된다

### B.2 ChatGPT Apps 실습

- ChatGPT에서 앱 디렉터리를 열고 앱 하나를 연결한다
- 권장 실습:
  1. 앱 디렉터리에서 GitHub나 문서 검색 계열 앱을 찾는다
  2. 연결 가능한 앱을 활성화한다
  3. 다음과 같이 요청한다

```text
연결된 앱을 사용해서 이 저장소 또는 연결된 문서에서 week3 실습과 관련된 정보를 찾아 요약해줘.
```

- 관찰 포인트
  - 앱이 어떤 정보를 검색·참조하는가
  - 단순 채팅과 무엇이 다른가
  - ChatGPT Apps가 MCP와 같은 표준인지, 아니면 제품 표면인지 구분할 수 있는가

### B.3 ChatGPT의 MCP 실습

- OpenAI 공식 문서 기준 ChatGPT Apps와 API 통합은 MCP 서버와 연결될 수 있다
- 수업에서는 구현보다 개념 구분에 집중한다
- 핵심 메시지:
  - ChatGPT Apps는 사용자 표면
  - MCP는 그 뒤쪽의 도구 연결 표준일 수 있다

### B.4 Codex에서의 Skills / Rules 실습

- Codex는 ChatGPT 계정으로 사용하는 코딩 에이전트이며, terminal, IDE, app, cloud 표면을 함께 제공한다
- 공식 안내 기준 Codex app은 skills, automations, git 기능을 제공한다
- 3주차 실습에서는 다음과 같이 수행한다

```text
이 저장소를 읽고 docs/notes.md를 요약해줘.
반드시 output/summary.md에 저장하고, 검증 항목 3개를 함께 적어줘.
```

- 그리고 규칙을 준 뒤 다시 비교한다

```text
다음 규칙을 지켜줘:
- 출력은 output/에 저장
- 검증 항목 3개 작성
- 불확실한 내용은 추측하지 말 것
이 규칙을 지키면서 docs/notes.md를 다시 요약해줘.
```

### B.5 Codex의 MCP 실습

- Codex 관련 공식 문서 네비게이션에는 MCP와 Skills가 별도 구성 항목으로 존재한다
- 수업에서는 다음 수준까지 실습하는 것을 목표로 한다
  - Codex가 MCP 서버를 쓰는 환경이라는 점을 이해한다
  - 같은 작업을 "도구 없이"와 "MCP 연결 후" 비교한다
  - 필요한 경우 API 쪽에서는 MCP server 문서와 Responses tool 문서를 참조한다

### B.6 Codex / ChatGPT에서 추가로 볼 것

- **Apps**
  - ChatGPT 안에서 기능을 붙이는 현재형 표면
- **Rules / Skills**
  - 반복 작업 규칙을 고정하는 계층
- **Automations**
  - 반복 작업을 자동 실행 흐름으로 옮기는 계층
- **Memory**
  - 지속 선호와 문맥을 유지하는 계층

### B.7 OpenAI 부록의 의도

- OpenAI 표면은 이름이 바뀌기 쉬우므로 학생이 가장 먼저 익혀야 할 것은 메뉴 이름이 아니라 구조임
- 이 부록의 핵심 대응 관계:
  - GitHub Copilot plugin ↔ ChatGPT Apps
  - GitHub Skills ↔ Codex skills / rules
  - GitHub hooks ↔ Codex automations
  - GitHub memory ↔ OpenAI memory
  - GitHub MCP ↔ OpenAI MCP integrations
- 목적은 특정 제품 조작법 암기가 아니라, **동일한 실습 구조를 다른 벤더 표면으로 번역하는 능력**을 기르는 것임

---

## 참고 자료

- GitHub Copilot agent mode: https://docs.github.com/en/copilot/how-tos/chat/asking-github-copilot-questions-in-your-ide
- GitHub MCP and coding agent: https://docs.github.com/en/copilot/concepts/agents/coding-agent/mcp-and-coding-agent
- GitHub Copilot CLI plugins overview: https://docs.github.com/copilot/concepts/agents/copilot-cli/about-cli-plugins
- GitHub Copilot CLI plugin creation: https://docs.github.com/copilot/how-tos/copilot-cli/customize-copilot/plugins-creating
- GitHub Copilot CLI plugin install: https://docs.github.com/en/enterprise-cloud%40latest/copilot/how-tos/copilot-cli/customize-copilot/plugins-finding-installing
- GitHub Agent Skills: https://docs.github.com/en/copilot/concepts/agents/about-agent-skills
- GitHub custom instructions: https://docs.github.com/en/copilot/how-tos/custom-instructions/add-repository-instructions
- Claude Code overview: https://docs.anthropic.com/en/docs/claude-code/overview
- Claude Code IDE integration: https://docs.anthropic.com/en/docs/claude-code/ide-integrations
- Claude Code MCP: https://docs.anthropic.com/en/docs/claude-code/mcp
- Claude Code Skills: https://docs.anthropic.com/en/docs/claude-code/slash-commands
- Claude Code plugins: https://code.claude.com/docs/en/plugins
- Claude Code settings: https://docs.anthropic.com/en/docs/claude-code/settings
- Claude Code hooks: https://docs.anthropic.com/en/docs/claude-code/hooks
- Claude Code memory: https://docs.anthropic.com/en/docs/claude-code/memory
- OpenAI MCP docs: https://developers.openai.com/api/docs/mcp
- OpenAI tools docs: https://developers.openai.com/api/docs/guides/tools
- Apps in ChatGPT: https://help.openai.com/en/articles/11487775-connectors-in-chatgpt
- Codex in ChatGPT plans: https://help.openai.com/en/articles/11369540/
- Introducing the Codex app: https://openai.com/index/introducing-the-codex-app/

---

## 다음 주 예고

- 4주차에서는 MCP 서버를 더 실전적으로 다룬다
- 인증, 실패 처리, 로깅, 테스트 가능한 구조를 갖춘 서버 설계로 발전시킨다
- 단순 연결에서 끝나지 않고, 실제 운영 가능한 도구 계층을 만드는 방향으로 확장한다
