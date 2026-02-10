# Lectures: GitHub Copilot으로 AI 에이전트 만들기

**핸즈온 워크숍 시리즈 (6회)**

---

## 개요

이 강의는 GitHub Copilot의 Agent Mode, MCP, Skills를 활용하여 실제 동작하는 AI 에이전트를 만드는 실습 중심 과정이다. `docs/`의 12장 교재가 이론과 프레임워크 비교에 초점을 둔다면, 이 강의는 **"지금 바로 만들어 본다"**에 집중한다.

## 수강 전 준비

| 항목 | 설명 |
|-----|------|
| VS Code | 최신 버전 (1.102+), Insiders 권장 |
| GitHub 계정 | [Student Developer Pack](https://education.github.com/pack) 인증 → Copilot Pro 무료 |
| GitHub Copilot 확장 | VS Code에서 `GitHub Copilot` + `GitHub Copilot Chat` 설치 |
| Python 3.10+ | MCP 서버 개발용 |
| Node.js 18+ | MCP 서버 실행용 (`npx`) |
| Git | 기본 사용법 숙지 |

## 강의 구성

```
[1] Agent Mode ─── Copilot이 코드를 자율로 짜는 방식을 체험한다
       │
[2] MCP 연결 ───── 기존 MCP 서버를 Copilot에 연결해 도구를 확장한다
       │
[3] MCP 제작 ───── 나만의 MCP 서버를 만들어 Copilot에 새 능력을 부여한다
       │
[4] Skills ─────── SKILL.md로 Copilot에 도메인 지식과 절차를 가르친다
       │
[5] 통합 프로젝트 ─ MCP + Skills를 조합해 자동화 에이전트를 완성한다
       │
[6] 팀 워크플로우 ─ Coding Agent로 이슈→PR→리뷰 파이프라인을 구축한다
```

| 회차 | 파일 | 주제 | 핵심 산출물 |
|-----|------|------|-----------|
| 1 | [lecture-01.md](lecture-01.md) | Copilot Agent Mode 실전 | 자연어로 만든 웹앱 |
| 2 | [lecture-02.md](lecture-02.md) | MCP 이해와 기존 서버 연결 | `.vscode/mcp.json` + 도구 호출 |
| 3 | [lecture-03.md](lecture-03.md) | 나만의 MCP 서버 만들기 | Python MCP 서버 1개 |
| 4 | [lecture-04.md](lecture-04.md) | Skills로 Copilot 커스터마이징 | SKILL.md 2개 이상 |
| 5 | [lecture-05.md](lecture-05.md) | MCP + Skills 통합 프로젝트 | 자동화 에이전트 1개 |
| 6 | [lecture-06.md](lecture-06.md) | Coding Agent와 팀 워크플로우 | Issue → 자동 PR → 리뷰 |

## docs/ 교재와의 관계

| 강의 | 관련 교재 장 |
|-----|------------|
| 1회 | 제1장 (AI 협업 개발) |
| 2-3회 | 제3-4장 (MCP 개념과 서버 구축) |
| 4회 | — (Skills는 교재 범위 밖, 강의 고유 내용) |
| 5회 | 제5장 (도구를 쓰는 에이전트) |
| 6회 | 제11장 (Human-in-the-loop) |

## 폴더 구조

```
lectures/
├── README.md              # 이 파일
├── lecture-01.md           # 1회차 강의안
├── lecture-02.md           # 2회차 강의안
├── lecture-03.md           # 3회차 강의안
├── lecture-04.md           # 4회차 강의안
├── lecture-05.md           # 5회차 강의안
└── lecture-06.md           # 6회차 강의안
```

실습 코드는 기존 `practice/` 폴더 구조를 따른다.
