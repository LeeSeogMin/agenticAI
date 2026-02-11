# 작업 체크리스트

## Phase 0: v1.0 초기 집필 — ✅ 완료

- [x] 프로젝트 초기 구조 설정 (2026-01-03)
- [x] v1.0 전 챕터 집필 (ch1-12, 12개 콘텐츠) (2026-01-03)
- [x] Multi-LLM 리뷰 스크립트 작성 (scripts/multi_llm_review.py)
- [x] contents.md v2.0 갱신 — 2026 기준 [신규] 항목 대거 추가 (2026-02-08)
- [x] CLAUDE.md 구조 개선 (2026-02-08)
- [x] GitHub Copilot 강의안 6회 작성 (lectures/) (2026-02-11)
- [x] 깃허브.md 학생용 가이드 작성 (2026-02-11)

---

## Phase 1: v2.0 본문 반영 — 미착수

각 챕터당 7단계 워크플로우(Planning → Research → Analysis → Implementation → Optimization → Verification → Conversion)를 수행한다. v1.0 본문을 기반으로 [신규] 항목을 추가하고 전체 분량을 보강한다.

### 검토만 (1개)

v1.0 본문이 완성도 높아 교정·일관성 점검만 수행한다.

#### ch2: 로컬 개발 환경과 재현 가능한 실행
- [ ] 본문 교정 및 일관성 점검
- [ ] 그래픽 (content/graphics/ch2/ — Mermaid)
- [ ] Multi-LLM 리뷰 (v2.0 기준)
- [ ] 최종 원고 확정 (docs/ch2.md)
> **참고**: 903줄, [신규] 항목 없음. 교정·서술 스타일 통일만 필요

### 소폭수정 (1개)

v1.0 본문 유지하면서 보강·교정한다.

#### ch9: GraphRAG/LightRAG/KAG
- [ ] 본문 보강 및 교정
- [ ] 그래픽 (content/graphics/ch9/ — Mermaid)
- [ ] Multi-LLM 리뷰 (v2.0 기준)
- [ ] 최종 원고 확정 (docs/ch9.md)
> **참고**: 250줄, [신규] 항목 없음. 분량 보강 + 서술 스타일 통일

### 대폭수정 (2개)

v1.0 본문에 [신규] 섹션 2-3개를 삽입한다.

#### ch3: MCP 개념과 도구/리소스 설계 (2025-11-25 스펙)
- [ ] 집필계획서 갱신 (schema/chap3.md — [신규] 반영)
- [ ] 리서치 (MCP 2025-11-25 스펙, 생태계, AAIF 거버넌스)
- [ ] [신규] MCP 2025 스펙 핵심 변경 섹션 추가
- [ ] [신규] MCP 생태계 현황 섹션 추가
- [ ] [신규] MCP 거버넌스 섹션 추가
- [ ] 실습 코드 갱신 (Tasks primitive 비동기 작업 포함)
- [ ] 그래픽 (content/graphics/ch3/ — Mermaid)
- [ ] Multi-LLM 리뷰
- [ ] 최종 원고 확정 (docs/ch3.md)
> **참고**: 628줄, 기존 본문 탄탄. [신규] 3개 섹션 삽입

#### ch11: Human-in-the-loop 워크플로우 설계
- [ ] 집필계획서 갱신 (schema/chap11.md)
- [ ] 리서치 (점진적 배포, 감사 로그)
- [ ] [신규] 점진적 배포 전략 섹션 추가
- [ ] [신규] 감사 로그 설계 섹션 추가
- [ ] 실습 코드 갱신 (Shadow Mode 단계 추가)
- [ ] 그래픽 (content/graphics/ch11/ — Mermaid)
- [ ] Multi-LLM 리뷰
- [ ] 최종 원고 확정 (docs/ch11.md)
> **참고**: 267줄, [신규] 2개 섹션 삽입

### 재작성 (8개)

v1.0 본문 기반이나 분량 부족 + [신규] 항목 다수로 전면 보강한다.

#### ch1: AI 협업이 바꾸는 개발 프로세스
- [ ] 집필계획서 갱신 (schema/chap1.md)
- [ ] 리서치 (에이전틱 코딩 도구, 바이브 코딩)
- [ ] [신규] 에이전틱 코딩 도구 전경 섹션 작성
- [ ] [신규] 바이브 코딩의 가능성과 한계 섹션 작성
- [ ] 기존 본문 대폭 보강 (현 77줄 → 목표 500줄+)
- [ ] 실습 코드 갱신
- [ ] 그래픽 (content/graphics/ch1/ — Mermaid)
- [ ] Multi-LLM 리뷰
- [ ] 최종 원고 확정 (docs/ch1.md)
> **참고**: 현재 77줄로 현저히 부족. 사실상 재작성

#### ch4: 첫 MCP 서버 만들기: 외부 API 래핑과 A2A 개요
- [ ] 집필계획서 갱신 (schema/chap4.md)
- [ ] 리서치 (OAuth 2.1, A2A v0.3, 프로토콜 비교)
- [ ] [신규] OAuth 2.1 인증 섹션 작성
- [ ] [신규] A2A 프로토콜 개요 섹션 작성
- [ ] [신규] 프로토콜 선택 기준 섹션 작성
- [ ] 실습 코드 갱신 (OAuth 인증 포함 MCP 서버)
- [ ] 그래픽 (content/graphics/ch4/ — Mermaid)
- [ ] Multi-LLM 리뷰
- [ ] 최종 원고 확정 (docs/ch4.md)
> **참고**: 242줄 + [신규] 3개. A2A는 완전 신규 주제

#### ch5: LangChain과 도구 호출의 진화
- [ ] 집필계획서 갱신 (schema/chap5.md)
- [ ] 리서치 (Structured Outputs, Tool Search, OpenAI Agents SDK)
- [ ] [신규] 구조화된 출력 섹션 작성
- [ ] [신규] 고급 도구 호출 패턴 섹션 작성
- [ ] [신규] OpenAI Agents SDK 소개 섹션 작성
- [ ] 실습 코드 갱신 (LangChain + Agents SDK 비교)
- [ ] 그래픽 (content/graphics/ch5/ — Mermaid)
- [ ] Multi-LLM 리뷰
- [ ] 최종 원고 확정 (docs/ch5.md)
> **참고**: 209줄 + [신규] 3개. OpenAI Agents SDK는 신규 주제

#### ch6: LangGraph 1.0으로 상태·분기·반복을 제어한다
- [ ] 집필계획서 갱신 (schema/chap6.md)
- [ ] 리서치 (LangGraph 1.0, LangSmith 통합, 장기 메모리)
- [ ] [신규] LangGraph 1.0 섹션 작성
- [ ] [신규] LangSmith 통합 섹션 작성
- [ ] [신규] LangGraph + 장기 메모리 섹션 작성
- [ ] 실습 코드 갱신 (LangGraph 1.0 + LangSmith 트레이스)
- [ ] 그래픽 (content/graphics/ch6/ — Mermaid)
- [ ] Multi-LLM 리뷰
- [ ] 최종 원고 확정 (docs/ch6.md)
> **참고**: 214줄 + [신규] 3개

#### ch7: 멀티에이전트 시스템: 프레임워크 비교와 A2A 실전
- [ ] 집필계획서 갱신 (schema/chap7.md)
- [ ] 리서치 (프레임워크 비교 2026, A2A 실전, 로우코드/노코드)
- [ ] [신규] 프레임워크 비교(2026 기준) 섹션 작성
- [ ] [신규] A2A 실전 섹션 작성
- [ ] [신규] 로우코드/노코드 대안 섹션 작성
- [ ] 실습 코드 갱신 (LangGraph + Agents SDK/ADK 비교)
- [ ] 그래픽 (content/graphics/ch7/ — Mermaid)
- [ ] Multi-LLM 리뷰
- [ ] 최종 원고 확정 (docs/ch7.md)
> **참고**: 255줄 + [신규] 3개

#### ch8: RAG의 기본과 에이전트 메모리 아키텍처
- [ ] 집필계획서 갱신 (schema/chap8.md)
- [ ] 리서치 (에이전트 메모리 분류, 구현 전략, RAG vs 메모리)
- [ ] [신규] 에이전트 메모리 분류 체계 섹션 작성
- [ ] [신규] 메모리 구현 전략 섹션 작성
- [ ] [신규] RAG vs 메모리 섹션 작성
- [ ] 실습 코드 갱신 (Episodic Memory 추가)
- [ ] 그래픽 (content/graphics/ch8/ — Mermaid)
- [ ] Multi-LLM 리뷰
- [ ] 최종 원고 확정 (docs/ch8.md)
> **참고**: 265줄 + [신규] 3개. 에이전트 메모리는 신규 주제

#### ch10: 에이전트 보안과 신뢰성: 검증에서 가드레일까지
- [ ] 집필계획서 갱신 (schema/chap10.md)
- [ ] 리서치 (OWASP AI Agent Security, 가드레일, 평가 프레임워크)
- [ ] [신규] OWASP AI Agent Security Top 10 섹션 작성
- [ ] [신규] 다층 가드레일 아키텍처 섹션 작성
- [ ] [신규] 프레임워크 내장 가드레일 섹션 작성
- [ ] [신규] 에이전트 평가 프레임워크 섹션 작성
- [ ] 실습 코드 갱신 (입력/출력 가드레일 적용)
- [ ] 그래픽 (content/graphics/ch10/ — Mermaid)
- [ ] Multi-LLM 리뷰
- [ ] 최종 원고 확정 (docs/ch10.md)
> **참고**: 282줄 + [신규] 4개 (가장 많은 신규 항목)

#### ch12: 배포·관측·비용 최적화
- [ ] 집필계획서 갱신 (schema/chap12.md)
- [ ] 리서치 (관측성 플랫폼, 비용 최적화, MCP Registry)
- [ ] [신규] 관측성 플랫폼 비교 섹션 작성
- [ ] [신규] 비용 최적화 설계 섹션 작성
- [ ] [신규] MCP Registry와 서버 발견 섹션 작성
- [ ] 실습 코드 갱신 (관측성 도구 연동)
- [ ] 그래픽 (content/graphics/ch12/ — Mermaid)
- [ ] Multi-LLM 리뷰
- [ ] 최종 원고 확정 (docs/ch12.md)
> **참고**: 288줄 + [신규] 3개

---

## Phase 2: 통합 검수 — 미착수

- [ ] 전 챕터 일관성 검증 (용어, 서술체, 코드 스타일)
- [ ] checklists/book-progress.md 실제 데이터로 전면 업데이트
- [ ] 참고문헌 검증 (URL/DOI 유효성 확인)
- [ ] 실습 코드 크로스 플랫폼 검증 (macOS + Windows)
- [ ] 표/그림 번호 체계 전체 정합성 확인

---

## Phase 3: 발행 — 미착수

### 3-1. MS Word 변환
- [ ] ms-word/ 변환 시스템 구축
- [ ] ch1-ch12 전체 변환

### 3-2. Notion 발행
- [ ] Notion Integration 생성 및 API Key 발급
- [ ] Notion 데이터베이스 생성
- [ ] .env 파일 설정
- [ ] ch1-ch12 전체 발행

---

## Phase 4: 후속 작업

- [ ] 표지 및 머리말/서문 작성
- [ ] 색인(Index) 생성
- [ ] 최종 교정 및 통합 검수

---

## 범례

| 기호 | 의미 |
|------|------|
| [x] | 완료 |
| [ ] | 미완료 |
| [-] | 해당 없음 |

---

**마지막 업데이트**: 2026-02-11
**현재 Phase**: Phase 1 미착수
**다음 작업**: ch1 재작성 또는 ch2 검토
