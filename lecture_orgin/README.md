# Lectures: GitHub Copilot으로 AI 에이전트 만들기

**15주 정규과목 (2026년 1학기)**

---

## 개요

이 강의는 GitHub Copilot의 Agent Mode, MCP, Skills를 활용하여 실제 동작하는 AI 에이전트를 만드는 실습 중심 과정이다. `docs/`의 12장 교재가 이론과 프레임워크 비교에 초점을 둔다면, 이 강의는 **"지금 바로 만들어 본다"**에 집중한다.

- **주당 3시간**: 1회차 강의(90분) + 2회차 실습(90분)
- **대상**: 기본 프로그래밍 경험이 있는 학부생
- **선수 조건**: Python 또는 JavaScript 기초 문법, VS Code 사용 경험

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
Phase 1. 기초 (1~4주)
[1] 환경 구축 ─── AI 코딩 도구 전경을 파악하고 실습 환경을 완성한다
       │
[2] Agent 기초 ── Copilot Agent Mode로 자연어 개발을 체험한다
       │
[3] Agent 심화 ── 한계를 인식하고 효과적인 활용 전략을 세운다
       │
[4] MCP 개념 ─── MCP 아키텍처와 생태계를 이해한다

Phase 2. MCP (5~7주)
[5] MCP 연결 ─── 기존 MCP 서버를 연결하여 Copilot을 확장한다
       │
[6] MCP 구현 ─── Python SDK로 MCP 서버를 직접 만든다
       │
[7] MCP 심화 ─── 에러 처리, 테스트, 배포까지 고려한다

[8] ────────── 중간고사 ──────────

Phase 3. Skills & 통합 (9~11주)
[9]  Skills 기초 ─ SKILL.md 구조를 익히고 기본 스킬을 작성한다
       │
[10] Skills 심화 ─ 설계 패턴을 적용하고 도메인 특화 스킬을 만든다
       │
[11] 통합 프로젝트  MCP + Skills로 자동화 에이전트를 완성한다

Phase 4. 팀 & 프로젝트 (12~15주)
[12] Coding Agent ─ 이슈→PR→리뷰 팀 워크플로우를 자동화한다
       │
[13] 프로젝트 설계 ─ 최종 프로젝트를 체계적으로 설계한다
       │
[14] 프로젝트 개발 ─ 구현을 완성하고 발표를 준비한다
       │
[15] 기말고사 + 발표
```

## 주차별 강의안

| 주차 | 파일 | 주제 | 핵심 산출물 |
|:---:|------|------|-----------|
| 1 | [week-01.md](week-01.md) | AI 코딩 도구 전경과 개발 환경 구축 | 환경 구축 + Copilot 첫 대화 |
| 2 | [week-02.md](week-02.md) | Copilot Agent Mode 기초 | 자연어로 만든 웹앱 |
| 3 | [week-03.md](week-03.md) | Copilot Agent Mode 심화 | 리팩토링 결과 + 한계 분석 |
| 4 | [week-04.md](week-04.md) | MCP 개념과 아키텍처 | MCP 아키텍처 다이어그램 + mcp.json |
| 5 | [week-05.md](week-05.md) | MCP 서버 연결과 활용 | MCP 서버 3종 연결 + 도구 호출 |
| 6 | [week-06.md](week-06.md) | MCP 서버 구현 기초 | 날씨 MCP 서버 |
| 7 | [week-07.md](week-07.md) | MCP 서버 심화 | 커스텀 MCP 서버 (자유 주제) |
| 8 | [week-08.md](week-08.md) | **중간고사** | 이론 40% + 실기 60% |
| 9 | [week-09.md](week-09.md) | Skills 기초 | SKILL.md 2개 |
| 10 | [week-10.md](week-10.md) | Skills 심화 | 도메인 특화 스킬 + 설계 보고서 |
| 11 | [week-11.md](week-11.md) | MCP + Skills 통합 프로젝트 | 자동화 에이전트 1개 |
| 12 | [week-12.md](week-12.md) | Coding Agent와 팀 워크플로우 | Issue → 자동 PR → 리뷰 |
| 13 | [week-13.md](week-13.md) | 최종 프로젝트 설계 | 프로젝트 설계서 |
| 14 | [week-14.md](week-14.md) | 프로젝트 개발과 중간 점검 | 코드 최종 커밋 + 발표 자료 |
| 15 | [week-15.md](week-15.md) | 기말고사 + 프로젝트 발표 | 최종 발표 (10분 + 5분 Q&A) |

## docs/ 교재와의 관계

| 주차 | 관련 교재 장 |
|:---:|------------|
| 1 | 제1장 (AI 협업 개발 프로세스) |
| 2-3 | 제1장 (에이전틱 코딩, 바이브 코딩) |
| 4-5 | 제3장 (MCP 개념과 설계) |
| 6-7 | 제4장 (외부 API 래핑 MCP 서버) |
| 9-10 | — (Skills는 교재 범위 밖, 강의 고유 내용) |
| 11 | 제5장 (도구를 쓰는 에이전트) |
| 12 | 제11장 (Human-in-the-loop 워크플로우) |

## 폴더 구조

```
lectures/
├── syllabus.md              # 15주 강의계획서
├── README.md                # 이 파일
├── plan.md                  # 개편 계획안
├── week-01.md ~ week-15.md  # 주차별 강의안
└── _archive/
    └── lecture-01.md ~ lecture-06.md  # 기존 6회 워크숍 보관
```

실습 코드는 기존 `practice/` 폴더 구조를 따른다.
