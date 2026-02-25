# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

# 강의자료 작성 프로젝트 (GitHub Copilot 기반 AI 에이전트 과목)

6개 전문 에이전트(@planner, @researcher, @writer, @coder, @reviewer, @graphic)가 협력하여 15주 정규과목 강의안을 작성·검증하는 시스템이다. `docs/`의 12장 교재는 이론 참고 자료로 병행 관리한다.

---

## 빠른 시작

### 강의안 작성 명령
```bash
# "N주차 작성" → 리서치 → 강의안 집필 → 검토 (lectures/week-{NN}.md)
# "N주차 검토" → 기존 강의안 일관성 검증 + 품질 리뷰
# "N주차 실습 코드" → 해당 주차 실습 코드 작성/검증
```

### 교재 챕터 관련 (보조)
```bash
# "N장 작성" → 교재 집필 워크플로우 (계획→조사→집필→검토→Word변환)
# "N장 검토" → 일관성 검증 + 품질 리뷰 + Word 변환
# "N장 변환" → docs/ch{N}.md → Word
```

### Python 실습 코드 실행
```bash
cd practice/chapter{N}
python3 -m venv venv && source venv/bin/activate
pip install -r code/requirements.txt
python3 code/{N}-{M}-{주제}.py
# 산출물 → practice/chapter{N}/data/output/
```

### MS Word 변환 (교재용)
```bash
cd ms-word && npm install              # 최초 1회
npm run convert:chapter {N}            # 개별 챕터 (docs/ch{N}.md → ms-word/output/ch{NN}.docx)
npm run convert:all                    # 전체 일괄
```

### 코드 품질
```bash
black practice/chapter{N}/code/        # 포매팅
flake8 practice/chapter{N}/code/       # 린트
pytest                                 # 테스트 (루트에서)
```

---

## 강의 구조 (15주 4페이즈)

```
Phase 1. 기초 (1~4주)
[1] 환경 구축 ─── AI 코딩 도구 전경, VS Code + Copilot 설정
[2] Agent 기초 ── Copilot Agent Mode로 자연어 개발 체험
[3] Agent 심화 ── 한계 인식, 효과적 활용 전략
[4] MCP 개념 ─── MCP 아키텍처와 생태계 이해

Phase 2. MCP (5~7주)
[5] MCP 연결 ─── 기존 MCP 서버 연결, Copilot 확장
[6] MCP 구현 ─── Python SDK로 MCP 서버 직접 구현
[7] MCP 심화 ─── 에러 처리, 테스트, 배포

[8] ────────── 중간고사 ──────────

Phase 3. Skills & 통합 (9~11주)
[9]  Skills 기초 ─ SKILL.md 구조, 기본 스킬 작성
[10] Skills 심화 ─ 설계 패턴, 도메인 특화 스킬
[11] 통합 프로젝트  MCP + Skills 자동화 에이전트 완성

Phase 4. 팀 & 프로젝트 (12~15주)
[12] Coding Agent ─ 이슈→PR→리뷰 팀 워크플로우 자동화
[13] 프로젝트 설계 ─ 최종 프로젝트 설계서 작성
[14] 프로젝트 개발 ─ 구현 완성, 발표 준비
[15] 기말고사 + 발표
```

주차별 세부 내용은 `lectures/README.md` 참조. 강의계획서 원문은 `content/sources/syllabus.md`.

---

## 강의안 워크플로우

```
[1] 계획 ─── 주차별 학습목표·선수지식 설정
       │
[2] 리서치 ── 해당 주제 최신 자료 조사 ──▶ content/research/
       │
[3] 집필 ─── 1회차(강의) + 2회차(실습) ──▶ lectures/week-{NN}.md
       │
[4] 검토 ─── 일관성·정확성 검증, 교재(docs/) 교차 점검
```

### 주차 파일 템플릿

```markdown
# N주차. {제목}

> **1회차** (강의 90분): {강의 주제 한줄}
> **2회차** (실습 90분): {실습 주제 한줄}

---

## 학습목표
## 선수지식

---

## 1회차: 강의

### N.1 {섹션}
### N.2 {섹션}
### N.3 {섹션}

---

## 2회차: 실습

### 실습 1: {제목}
### 실습 2: {제목}

---

## 과제
## 핵심 정리
## 참고 자료
## 다음 주 예고
```

---

## 교재 워크플로우 (보조, 7단계)

```
[1] Planning ─── @planner ────▶ schema/chap{N}.md
[2] Research ─── @researcher ──▶ content/research/
[3] Analysis ─── 정보 구조화
[4] Implementation ── @writer + @coder + @graphic ──▶ content/drafts/ + practice/
[5] Optimization ── 일관성 검증
[6] Verification ── @reviewer ──▶ docs/ch{N}.md (최종본)
[7] Conversion ── MS Word ────▶ ms-word/output/ch{N:02}.docx
```

---

## 핵심 원칙 (절대 규칙)

### 실제 실행만
- 모든 코드는 실제 실행하여 결과 획득. 변경한 스크립트는 최소 1회 실행 검증.
- **금지**: 더미 데이터, "예시 출력", 가상 결과값, 허구의 참고문헌/URL/DOI

### 코드 제시 원칙
- 강의안 본문: **핵심 코드 3-5줄만** 인라인 포함
- 전체 코드: `practice/chapter{N}/code/{N}-{M}-{주제}.py`
- 참조 형식: `_전체 코드는 practice/chapter{N}/code/{파일명}.py 참고_`

### 크로스 플랫폼
- 경로: `pathlib.Path` 또는 `os.path.join()` 사용. 하드코딩 금지, 상대 경로 우선.
- 코드 내 기준점: `Path(__file__).parent.parent / "data"`

### 참고문헌
- 형식: `저자명. (연도). 논문제목. *저널명*. URL/DOI`
- 검증 가능한 실제 문헌만 사용

---

## 에이전트 라우팅

| 에이전트 | 트리거 키워드 | 도구 | 출력 위치 |
|---------|-------------|------|----------|
| @planner | 계획, 스키마, 주차 설계 | Plan Mode | `schema/` 또는 `lectures/plan.md` |
| @researcher | 조사, 리서치, 참고문헌 | WebSearch, WebFetch | `content/research/` |
| @writer | 작성, 초안, 강의안, 본문 | Read, Write, Edit | `lectures/week-{NN}.md` 또는 `content/drafts/` |
| @coder | 코드, 실습, 예제, 구현 | Write, Bash | `practice/chapter{N}/code/` |
| @reviewer | 검토, 리뷰, 피드백 | Read, Grep | `lectures/` 또는 `docs/` |
| @graphic | 다이어그램, 플로우차트 | Mermaid, Write | `content/graphics/` |

에이전트별 상세 역할·템플릿은 `.claude/agents/*.md` 참조. @planner는 반드시 `EnterPlanMode` → 탐색 → `ExitPlanMode` 순서로 작업한다.

---

## 작성 스타일

### 문체
- 격식체 평서문 ('이다', '한다'). 개조식 금지 (예외: 표, 학습목표, 핵심정리, 과제)
- 전문 용어 영문 병기: "모델 컨텍스트 프로토콜(MCP)"
- 순서: "왜 필요한가 → 어떻게 동작하는가"

### 수식 (Unicode 인라인만)
```
✅ Yᵢₜ = αᵢ + λₜ + δ·Dᵢₜ + εᵢₜ
❌ $Y_{it} = \alpha_i$ (LaTeX 금지)
```
첨자: ₀₁₂₃₄₅₆₇₈₉ₐₑᵢⱼₖₙₒₓ / ⁰¹²³⁴⁵⁶⁷⁸⁹ᵃᵇᶜᵈᵉ  |  그리스: αβγδεζηθλμπρστφχψω

### 표/그림 번호
- 표 제목: 표 **위** (`**표 2.1** 제목`), 그림 제목: 그림 **아래** (`**그림 3.2** 제목`)
- 강의안 번호 체계: `{주차}.{순번}` (예: 표 3.1, 그림 5.2)
- 교재 번호 체계: `{장}.{순번}` (예: 표 2.1, 그림 3.2)

### 코드 스타일
- Python 3.10+, PEP 8, 한국어 주석/docstring
- 재현성: 시드값 고정 필수 (`random.seed()`, `np.random.seed()`)

---

## 강의안-교재 연관 매핑

| 주차 | 강의 주제 | 관련 교재 장 |
|:---:|----------|------------|
| 1 | AI 코딩 도구 전경 + 환경 구축 | 제1장 |
| 2-3 | Copilot Agent Mode 기초/심화 | 제1장 |
| 4-5 | MCP 개념, 서버 연결 | 제3장 |
| 6-7 | MCP 서버 구현/심화 | 제4장 |
| 9-10 | Skills 기초/심화 | — (강의 고유) |
| 11 | MCP + Skills 통합 | 제5장 |
| 12 | Coding Agent, 팀 워크플로우 | 제11장 |

---

## 폴더 구조

```
agenticAI/
├── lectures/                    # ★ 주 작업 영역
│   ├── README.md                # 15주 강의 개요
│   ├── plan.md                  # 개편 계획안
│   ├── syllabus.md              # 강의계획서
│   ├── week-01.md ~ week-15.md  # 주차별 강의안
│   └── _archive/                # 기존 6회 워크숍 보관
├── docs/                        # 교재 최종 원고 (12장, 보조 참조)
│   ├── ch1.md ~ ch12.md
│   └── sample.md                # 본문 서술 표준
├── content/
│   ├── sources/syllabus.md      # 강의계획서 원문
│   ├── research/                # 리서치 자료
│   ├── drafts/                  # 교재 초안
│   ├── graphics/                # Mermaid 다이어그램
│   └── reviews/                 # Multi-LLM 리뷰 결과
├── schema/                      # 교재 집필계획서
├── practice/chapter{N}/         # 실습 코드 + 산출물
├── ms-word/                     # Word 변환 도구
├── contents.md                  # 교재 목차 및 집필 방향
├── 깃허브.md                     # 학생용 GitHub Copilot 가이드
└── todo.md                      # 작업 체크리스트
```

---

## 환경 변수 (.env)

```bash
# Multi-LLM 리뷰 (교재 검수 시)
OPENAI_API_KEY=...
ANTHROPIC_API_KEY=...
GROK_API_KEY=...                    # 또는 XAI_API_KEY

# 모델 설정
OPENAI_MODEL=gpt-4o
CLAUDE_MODEL=claude-sonnet-4-20250514
GROK_MODEL=grok-4-1-fast-reasoning

# 선택
FIRECRAWL_API_KEY=...
```

---

## 진행 관리

- `todo.md`: 전체 프로젝트 진행상태 (강의안 + 교재)
- `lectures/plan.md`: 15주 강의 개편 계획 및 완료 체크
- 작업 완료 시 해당 칸을 갱신한다.

---

## 필수 참조 문서

| 문서 | 용도 | 참조 시점 |
|-----|------|----------|
| `lectures/README.md` | 15주 강의 구조, 주차별 링크, 교재 연관 매핑 | 강의안 작성·검토 |
| `lectures/plan.md` | 개편 계획, 콘텐츠 소스 매핑, 파일 템플릿 | 강의안 작성 |
| `contents.md` | 교재 장별 절 구성, 강화 항목, 요구사항 | 교재 관련 작업 |
| `docs/sample.md` | 본문 서술 형식, 코드 제시 표준, 표/그림/수식 규칙 | 집필·검토 |
| `깃허브.md` | 학생용 GitHub Copilot/MCP/Skills 가이드 | 강의안 실습 설계 |
| `.claude/agents/*.md` | 에이전트별 상세 역할, 도구, 출력 템플릿 | 에이전트 전환 시 |

---

**마지막 업데이트**: 2026-02-25
**템플릿 버전**: 2.0
