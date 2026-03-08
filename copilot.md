# GitHub Copilot 사용 설명서

이 문서는 VS Code에서 GitHub Copilot을 활용하는 방법을 안내합니다. 아래 캡처 이미지를 참고하여 주요 기능과 메뉴를 설명합니다.

---

## 1. Copilot Chat 세션 시작

- **새 챗 세션(New Chat Session)**: Ctrl+N 단축키로 새로운 Copilot Chat 세션을 시작할 수 있습니다.
  **Continue In**: 챗 세션을 실행할 환경을 선택할 수 있습니다.
  - Local(로컬): 현재 PC에서 직접 Copilot 챗을 실행합니다. 빠른 응답과 파일 접근이 가능합니다.
  - Background(@cli): 명령줄 환경에서 Copilot을 실행합니다. 백그라운드 작업이나 자동화에 적합합니다.
  - Cloud(@cloud): 클라우드 서버에서 Copilot 챗을 실행합니다. 대용량 작업, 서버 리소스 활용, 원격 협업에 유리합니다.

---

## 2. 에이전트 및 역할 선택

- **Agent 메뉴**: Ctrl+Shift+I로 Copilot의 역할을 선택할 수 있습니다.
  - coder: 코드 작성
  - graphic: 그래픽/다이어그램 생성
  - planner: 집필 계획
  - researcher: 자료 조사
  - reviewer: 품질 검토
  - writer: 원고 작성
- **Plan/Ask**: 계획 수립 또는 질문 모드로 전환 가능
- **커스텀 에이전트 구성**: 필요에 따라 직접 에이전트 역할을 추가/설정할 수 있습니다.

---

## 3. 도구(툴) 설정

- **Configure Tools**: Copilot이 사용할 수 있는 도구를 선택/해제할 수 있습니다.
  - 예시: edit(파일 편집), execute(코드 실행), search(검색), todo(할 일 관리), web(웹 정보 수집) 등
  - MCP 서버 기반의 확장 도구도 활성화 가능
- **도구 선택 후 OK 버튼으로 적용**

---

## 4. MCP 서버 및 확장 기능

- **MCP 서버 알림**: 새로운 MCP 서버(예: mcp-server-time, GitHub 등)가 활성화되면 알림이 표시됩니다.
- **자동 시작 옵션**: "Automatically start MCP servers" 체크박스로 서버 자동 실행 설정 가능

---

## 5. Copilot 모델 선택

- **모델 선택**: 하단 메뉴에서 GPT-4.1 등 Copilot이 사용할 모델을 선택할 수 있습니다.

---

## 6. 기타 기능

- **챗 세션 관리**: 여러 챗 세션을 동시에 운영 가능
- **도구/에이전트/모델 조합**: 작업 목적에 따라 자유롭게 조합하여 활용

---

## 참고

- Copilot은 코드 작성, 문서화, 그래픽 생성, 품질 검토 등 다양한 역할을 수행할 수 있습니다.
- 각 메뉴와 도구는 프로젝트 목적에 맞게 선택하여 사용하세요.

---