# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

# 대중 교재 집필 프로젝트 (강의계획서 기반)

6개 전문 에이전트(@planner, @researcher, @writer, @coder, @reviewer, @graphic)가 협력하여 강의계획서를 AI 에이전트 개발 교재(12장, 5파트)로 집필·검증·변환하는 시스템이다.

---

## 빠른 시작

### 챕터 작성 명령
```bash
# "N장 작성" → 1~7단계 전체 자동 수행 (계획→조사→집필→검토→Word변환)
# "N장 검토" → 5~7단계 (일관성 검증 + 품질 리뷰 + Word 변환)
# "N장 변환" → 7단계만 (docs/ch{N}.md → Word)
```

### Python 실습 코드 실행
```bash
cd practice/chapter{N}
python3 -m venv venv && source venv/bin/activate
pip install -r code/requirements.txt
python3 code/{N}-{M}-{주제}.py
# 산출물 → practice/chapter{N}/data/output/
```

### MS Word 변환
```bash
cd ms-word && npm install              # 최초 1회
npm run convert:chapter {N}            # 개별 챕터 (docs/ch{N}.md → ms-word/output/ch{NN}.docx)
npm run convert:all                    # 전체 일괄
npm run create:book                    # 통합 도서
```

### Multi-LLM 품질 리뷰 (필수)
```bash
python3 scripts/multi_llm_review.py --chapter {N}
# 출력: content/reviews/ch{N}_review_YYYY-MM-DD.json
# .env에 OPENAI_API_KEY, ANTHROPIC_API_KEY, GROK_API_KEY 필요
```

### 코드 품질
```bash
black practice/chapter{N}/code/        # 포매팅
flake8 practice/chapter{N}/code/       # 린트
pytest                                 # 테스트 (루트에서)
```

---

## 7단계 워크플로우 (핵심 아키텍처)

```
[1] Planning ─── @planner (Plan Mode) ────▶ schema/chap{N}.md
        │
[2] Research ─── @researcher ─────────────▶ content/research/ch{N}-{section}-{topic}.md
        │
[3] Analysis ─── 정보 구조화 ──────────────▶ content/drafts/ch{N}-outline.md (선택)
        │
[4] Implementation ┬─ @coder ─────────────▶ practice/chapter{N}/code/{N}-{M}-{topic}.py
                   ├─ @writer ────────────▶ content/drafts/ch{N}-{section}-{topic}.md
                   └─ @graphic ───────────▶ content/graphics/ch{N}/fig-{N}-{seq}-{topic}.mmd
        │
[5] Optimization ── 일관성 검증
        │
[6] Verification ── @reviewer + Multi-LLM ▶ docs/ch{N}.md (최종본)
        │
[7] Conversion ─── MS Word ───────────────▶ ms-word/output/ch{N:02}.docx
```

**CRITICAL**: 4단계 완료 후 5-6-7단계는 자동 연속 수행. LLM 검토 없이 최종본 확정 불가.

### 단계별 데이터 흐름

1. **Planning**: `contents.md`의 장별 요구사항 → `schema/chap{N}.md` (절 구성, 코드 예제 계획, 분량 배분)
2. **Research**: 스키마 기반 조사 → `content/research/` (출처 명시 필수, 2023년 이후 자료 우선)
3. **Implementation**: 리서치 + 스키마 → 본문 초안 + 실습 코드 + 다이어그램 병렬 생성
4. **Verification**: 초안 + 코드 실행 결과 + `contents.md` 교차 점검 → `docs/ch{N}.md` 확정
5. **Conversion**: 확정 원고 → DOCX (A4, Malgun Gothic/Apple SD Gothic Neo, 1.6 줄간격)

---

## 핵심 원칙 (절대 규칙)

### 실제 실행만
- 모든 코드는 실제 실행하여 결과 획득. 변경한 스크립트는 최소 1회 실행 검증.
- **금지**: 더미 데이터, "예시 출력", 가상 결과값, 허구의 참고문헌/URL/DOI

### 코드 제시 원칙
- 본문: **핵심 코드 3-5줄만**
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
| @planner | 계획, 스키마, 집필계획서 | Plan Mode | `schema/chap{N}.md` |
| @researcher | 조사, 리서치, 참고문헌 | WebSearch, WebFetch | `content/research/` |
| @writer | 작성, 초안, 원고, 본문 | Read, Write, Edit | `content/drafts/` → `docs/` |
| @coder | 코드, 실습, 예제, 구현 | Write, Bash | `practice/chapter{N}/code/` |
| @reviewer | 검토, 리뷰, 피드백 | Read, Grep | `docs/ch{N}.md` |
| @graphic | 다이어그램, 플로우차트 | Mermaid, Write | `content/graphics/ch{N}/` |

에이전트별 상세 역할·템플릿은 `.claude/agents/*.md` 참조. @planner는 반드시 `EnterPlanMode` → 탐색 → `ExitPlanMode` 순서로 작업한다.

---

## 작성 스타일

### 문체
- 격식체 평서문 ('이다', '한다'). 개조식 금지 (예외: 표, 학습목표, 핵심정리)
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
- 번호 체계: `{장}.{순번}` (예: 표 2.1, 그림 3.2)

### 코드 스타일
- Python 3.10+, PEP 8, 한국어 주석/docstring
- 재현성: 시드값 고정 필수 (`random.seed()`, `np.random.seed()`)

---

## 도서 구조 (5파트 12장)

| 파트 | 챕터 | 주제 |
|-----|------|------|
| 1: AI 협업 기초 | 1-2 | 개발 프로세스 변화, 로컬 환경/재현 실행 |
| 2: MCP 도구 표준화 | 3-4 | MCP 개념/설계, 외부 API 래핑 서버 |
| 3: 에이전트 프레임워크 | 5-7 | LangChain, LangGraph, 멀티 에이전트 |
| 4: 지식 시스템/RAG | 8-9 | 기본 RAG, GraphRAG/LightRAG |
| 5: 신뢰성/운영/배포 | 10-12 | 할루시네이션 방지, Human-in-loop, 배포/비용 |

장별 세부 요구사항은 `contents.md` 참조. 각 장은 학습목표(2-4개), 의사결정 포인트, 실패 사례 ≥1개, 실습 코드 참조를 반드시 포함해야 한다.

---

## 환경 변수 (.env)

```bash
# Multi-LLM 리뷰 (필수)
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

- `checklists/book-progress.md`: 장별 7단계 진행상태 (R/W/C/G/V 플래그)
- 작업 완료 시 해당 칸을 갱신한다.

---

## 필수 참조 문서

| 문서 | 용도 | 참조 시점 |
|-----|------|----------|
| `AGENTS.md` | 절대 금지 규칙, 실행/산출물 관리, Word 변환 절차 | 모든 작업 |
| `contents.md` | 장별 절 구성, 강화 항목, 요구사항 | 계획·집필·검토 |
| `docs/sample.md` | 본문 서술 형식, 코드 제시 표준, 표/그림/수식 규칙 | 집필·검토 |
| `.claude/agents/*.md` | 에이전트별 상세 역할, 도구, 출력 템플릿 | 에이전트 전환 시 |

---

**마지막 업데이트**: 2026-02-08
**템플릿 버전**: 1.2
