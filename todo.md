# 작업 체크리스트

---

## A. 강의자료 (주 작업)

### A-0: 초기 구축 — ✅ 완료 (2026-02-11)

- [x] 종합계획안 작성 (lectures/plan.md)
- [x] syllabus.md 15주 정규과목 개편
- [x] README.md 업데이트
- [x] 기존 lecture-*.md → _archive/ 이동
- [x] week-01 ~ week-07 작성
- [x] week-08 (중간고사) 작성
- [x] week-09 ~ week-12 작성
- [x] week-13 ~ week-15 작성
- [x] 깃허브.md 학생용 가이드 작성

### A-1: 강의안 검토·보강 — ✅ 완료 (2026-02-25)

- [x] week-01 ~ week-15 일관성 검증 (용어 통일: 도구(Tool), URL 수정 18건)
- [x] 주차 간 선수지식-학습목표 연결 점검 (15주 전체 100% 정합)
- [x] 과제 난이도·분량 균형 점검 (Low→Medium→High 자연스러운 상승, 양호)
- [x] 참고 자료 URL 유효성 확인 (15개 깨진 URL 수정 완료)

### A-2: 실습 코드 정비 — ✅ 완료 (2026-02-25)

- [x] 강의 실습용 코드 practice/lectures/ 구조 생성 (주차별 디렉토리)
- [x] week-02 실습: Flask Todo 웹앱 가이드 (README.md + 체크리스트)
- [x] week-05 실습: MCP 서버 3종 연결 설정 (.vscode/mcp.json)
- [x] week-06 실습: 날씨 MCP 서버 구현 코드 (weather_server.py + .env.example + requirements.txt)
- [x] week-07 실습: 안정성 강화 날씨 서버 (검증/재시도/로깅 추가)
- [x] week-09 실습: SKILL.md 예제 2종 (code-review + api-docs + 번들 파일 + 테스트 앱)
- [x] week-11 실습: MCP + Skills 통합 에이전트 (mcp.json + weekly-report SKILL.md)
- [x] week-12 실습: Issue→PR 워크플로우 (copilot-setup-steps.yml + 이슈 템플릿 + mcp.json)

### A-3: 강의 보조자료 — 미착수

- [ ] 주차별 슬라이드 또는 요약 자료
- [ ] 중간고사 예시 문제 (week-08)
- [ ] 기말고사 예시 문제 (week-15)
- [ ] 프로젝트 설계서 템플릿 (week-13)
- [ ] 프로젝트 평가 루브릭

---

## B. 교재 (보조 작업)

### B-0: v1.0 초기 집필 — ✅ 완료

- [x] 프로젝트 초기 구조 설정 (2026-01-03)
- [x] v1.0 전 챕터 집필 (ch1-12) (2026-01-03)
- [x] Multi-LLM 리뷰 스크립트 작성 (scripts/multi_llm_review.py)
- [x] contents.md v2.0 갱신 (2026-02-08)
- [x] CLAUDE.md 구조 개선 (2026-02-08)

### B-1: v2.0 본문 반영 — ✅ 완료 (2026-02-11)

<details>
<summary>ch1~ch12 상세 (클릭하여 펼치기)</summary>

#### 검토만: ch2 (로컬 개발 환경)
- [x] 본문 교정 및 일관성 점검 (905줄)

#### 소폭수정: ch9 (GraphRAG/LightRAG/KAG)
- [x] 본문 보강 및 교정 (244줄)

#### 대폭수정: ch3 (MCP 개념/설계), ch11 (Human-in-the-loop)
- [x] ch3 최종 원고 확정 (705줄)
- [x] ch11 최종 원고 확정 (356줄)

#### 재작성: ch1, ch4, ch5, ch6, ch7, ch8, ch10, ch12
- [x] ch1 최종 원고 확정 (174줄)
- [x] ch4 최종 원고 확정 (319줄)
- [x] ch5 최종 원고 확정 (285줄)
- [x] ch6 최종 원고 확정 (316줄)
- [x] ch7 최종 원고 확정 (337줄)
- [x] ch8 최종 원고 확정 (340줄)
- [x] ch10 최종 원고 확정 (359줄)
- [x] ch12 최종 원고 확정 (403줄)

</details>

### B-2: 통합 검수 — 부분 완료 (2026-02-11)

- [x] 전 챕터 일관성 검증 (용어, 서술체, 코드 스타일)
- [x] 표/그림 번호 체계 정합성 확인
- [x] 형식 통일 (학습 목표, 핵심 정리, 선수 지식, 다음 장 예고)
- [x] v2-audit-report.md 생성
- [ ] 참고문헌 검증 (URL/DOI 유효성)
- [ ] 실습 코드 크로스 플랫폼 검증 (macOS + Windows)

### B-3: 품질 강화 — 미착수

- [ ] ch1-ch12 전체 Multi-LLM 리뷰 재수행
- [ ] 전 챕터 Mermaid 다이어그램 생성 (content/graphics/)

### B-4: 발행 — 미착수

- [ ] ch1-ch12 MS Word 변환 (ms-word/)
- [ ] Notion 발행 (Integration + DB + 전체 발행)

### B-5: 후속 작업 — 미착수

- [ ] 표지 및 머리말/서문 작성
- [ ] 색인(Index) 생성
- [ ] 실습 코드 v2.0 동기화 검증
- [ ] 최종 교정

---

## 범례

| 기호 | 의미 |
|------|------|
| [x] | 완료 |
| [ ] | 미완료 |

---

**마지막 업데이트**: 2026-02-25
**현재 우선순위**: B-3 (교재 품질 강화) 또는 추가 강의 보강
**완료**: A-1 (강의안 검토·보강) + A-2 (실습 코드 정비)
