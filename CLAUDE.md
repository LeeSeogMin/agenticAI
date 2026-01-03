# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

# 대중 교재 집필 프로젝트 (강의계획서 기반)

6개 전문 에이전트가 협력하여 강의계획서를 대중 교재로 집필·검증·변환하는 워크플로우 템플릿.

---

## 빠른 시작

### 챕터 작성 명령
```bash
# "N장 작성" → 1~7단계 전체 자동 수행 (계획→조사→집필→검토→Word변환)
# "N장 검토" → 5~7단계 (일관성 검증 + 품질 리뷰 + Word 변환)
# "N장 변환" → 7단계만 (docs/ch{N}.md → Word)
```

### MS Word 변환
```bash
cd ms-word && npm install              # 최초 1회
npm run convert:chapter 2              # 개별 챕터
npm run convert:all                    # 전체 일괄
npm run create:book                    # 통합 도서
```

### Multi-LLM 품질 리뷰 (필수)
```bash
python3 scripts/multi_llm_review.py --chapter {N}
# 출력: content/reviews/ch{N}_review_YYYY-MM-DD.json
```

### Python 실습 코드 실행
```bash
cd practice/chapter{N}
python3 -m venv venv && source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r code/requirements.txt
python3 code/{N}-{M}-{주제}.py
```

---

## 7단계 워크플로우 (핵심 아키텍처)

```
[1] Planning ─── @planner (Plan Mode) ────▶ schema/chap{N}.md
        │
[2] Research ─── @researcher ─────────────▶ content/research/
        │
[3] Analysis ─── 정보 구조화
        │
[4] Implementation ┬─ @coder ─────────────▶ practice/chapter{N}/code/
                   ├─ @writer ────────────▶ content/drafts/
                   └─ @graphic ───────────▶ content/graphics/
        │
[5] Optimization ── 일관성 검증
        │
[6] Verification ── @reviewer + Multi-LLM ▶ docs/ch{N}.md (최종본)
        │
[7] Conversion ─── MS Word ───────────────▶ ms-word/output/ch{N:02}.docx
```

**CRITICAL**: 4단계 완료 후 5-6-7단계는 자동 연속 수행.

---

## 핵심 원칙 (절대 규칙)

### 실제 실행만
- 모든 코드는 실제 실행하여 결과 획득
- **금지**: 더미 데이터, "예시 출력", 가상 결과값

### 코드 제시 원칙
- 본문: **핵심 코드 3-5줄만**
- 전체 코드: `practice/chapter{N}/code/{N}-{M}-{주제}.py`
- 참조 형식: `_전체 코드는 practice/chapter{N}/code/{파일명}.py 참고_`

### 크로스 플랫폼
- 경로: `pathlib.Path` 또는 `os.path.join()` 사용
- 하드코딩 금지, 상대 경로 우선

### 참고문헌
- 허구의 참고문헌 절대 금지
- 형식: `저자명. (연도). 논문제목. *저널명*. URL/DOI`

---

## 프로젝트 구조

```
project/
├── CLAUDE.md              # 이 파일
├── AGENTS.md              # 상세 운영 규칙 (필수 참조)
├── contents.md            # 목차 및 집필 방향 (필수 참조)
├── docs/
│   ├── sample.md          # 집필 표준 문서 (필수 참조)
│   └── ch{N}.md           # 최종 완성 원고
├── schema/chap{N}.md      # 집필계획서
├── content/
│   ├── research/          # 리서치 결과
│   ├── drafts/            # 원고 초안
│   ├── graphics/          # 시각자료 (Mermaid 등)
│   └── reviews/           # LLM 리뷰 결과 JSON
├── practice/chapter{N}/
│   ├── code/              # 실행 가능한 전체 코드 + requirements.txt
│   └── data/output/       # 실습 산출물
├── ms-word/
│   ├── src/               # 변환 스크립트 (Node.js)
│   ├── config/            # 설정 (formatting-standards.js, book-metadata.json)
│   └── output/            # 생성된 .docx 파일
├── scripts/
│   └── multi_llm_review.py  # 3개 LLM(OpenAI/Claude/Grok) 품질 검증
├── .claude/agents/        # 6개 에이전트 정의
│   ├── planner.md         # Plan Mode 사용
│   ├── researcher.md      # WebSearch, WebFetch
│   ├── writer.md          # Read, Write, Edit
│   ├── coder.md           # Write, Bash
│   ├── reviewer.md        # Read, Grep
│   └── graphic.md         # Mermaid, Write
└── checklists/            # 진행 체크리스트
```

---

## 에이전트 라우팅

| 에이전트 | 트리거 키워드 | 출력 위치 |
|---------|-------------|----------|
| @planner | 계획, 스키마, 집필계획서 | `schema/chap{N}.md` |
| @researcher | 조사, 리서치, 참고문헌 | `content/research/` |
| @writer | 작성, 초안, 원고, 본문 | `content/drafts/` → `docs/` |
| @coder | 코드, 실습, 예제, 구현 | `practice/chapter{N}/code/` |
| @reviewer | 검토, 리뷰, 피드백 | `docs/ch{N}.md` |
| @graphic | 다이어그램, 플로우차트 | `content/graphics/` |

---

## 작성 스타일

### 문체
- 격식체 평서문 ('이다', '한다')
- 개조식 금지 (예외: 표, 학습목표, 핵심정리)
- 전문 용어 영문 병기: "모델 컨텍스트 프로토콜(MCP)"

### 수식 (Unicode 인라인만)
```
✅ Yᵢₜ = αᵢ + λₜ + δ·Dᵢₜ + εᵢₜ
❌ $Y_{it} = \alpha_i$ (LaTeX 금지)
```

### 표/그림 번호
- 표 제목: 표 위 (`**표 2.1** 제목`)
- 그림 제목: 그림 아래 (`**그림 3.2** 제목`)

### 코드 스타일
- Python 3.10+, PEP 8 준수
- 한국어 주석/docstring

---

## 환경 변수 (.env)

```bash
# Multi-LLM 리뷰 (필수)
OPENAI_API_KEY=...
ANTHROPIC_API_KEY=...
GROK_API_KEY=...       # 또는 XAI_API_KEY

# 선택
FIRECRAWL_API_KEY=...
```

---

## 필수 참조 문서

| 문서 | 용도 |
|-----|------|
| `AGENTS.md` | 상세 운영 규칙, 에이전트별 도구/출력 규칙 |
| `contents.md` | 목차 구성, 장별 강화 항목/요구사항 |
| `docs/sample.md` | 본문 서술 형식, 코드 제시 표준 |
| `.claude/agents/*.md` | 에이전트별 상세 역할 및 템플릿 |

---

**마지막 업데이트**: 2026-01-02
**템플릿 버전**: 1.1
