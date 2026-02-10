# 교재 목차 및 집필 방향 (대중 교재)

이 저장소는 `content/sources/syllabus.md`의 강의계획서를 기반으로, 실무 입문자가 따라올 수 있는 **대중적인 교재(텍스트북)**를 집필·검증·변환하기 위한 워크플로우 템플릿이다. 원고 작성 표준은 `docs/sample.md`를 따른다.

---

## 0) 교재의 목표와 독자

- 목표: AI 코딩 도구(Copilot, Claude Code, Cursor 등)와 MCP/A2A 기반 도구·에이전트 통합을 활용해, "설계·검증·운영"까지 이어지는 에이전트 개발 흐름을 학습자가 실제로 재현하게 한다.
- 독자: 기본 프로그래밍 경험이 있는 학부 고학년/대학원생, 또는 현업 주니어 개발자.
- 전제: 파이썬 개발 경험이 깊지 않아도 따라올 수 있도록, 코드보다 "문제 설정 → 설계 판단 → 검증"을 우선한다.
- 기준 시점: 2026년 상반기. MCP 2025-11-25 스펙, A2A v0.3, LangGraph 1.0, OpenAI Agents SDK, Google ADK 기준.

---

## 1) 집필 공통 규칙 (필수)

1. 본문에는 **핵심 코드만 3–5줄** 포함하고, 전체 코드는 `practice/`로 분리한다.
2. 본문 수치/표/그래프/출력/산출물 목록은 **실제 실행 결과만** 사용한다.
3. 플랫폼 종속 경로·명령어를 하드코딩하지 않는다(상대 경로 + `pathlib.Path`).
4. 참고문헌은 검증 가능한 실재 자료만 인용한다.
5. 본문은 서술형을 기본으로 하되, 학습목표/핵심정리/표/코드 블록은 예외로 한다.

---

## 2) 산출물(폴더) 규칙

- 최종 원고: `docs/ch{N}.md`
- 집필계획서: `schema/chap{N}.md`
- 초안/리서치/그래픽/리뷰: `content/`
- 실습 코드: `practice/chapter{N}/code/`
- 실습 산출물: `practice/chapter{N}/data/output/`
- Word 변환: `ms-word/output/ch{N:02}.docx` (예: `ch01.docx`)

---

## 3) 교재 구성(Parts/Chapters)

아래는 강의(16주) 흐름을 "학습자가 혼자서도 완주 가능한 장(Chapter)" 단위로 재구성한 목차이다. 각 장은 "왜 필요한가 → 무엇을 결정해야 하는가 → 최소 구현 → 검증/확장" 순서로 전개한다.

### Part 1. AI 협업 개발의 출발점

#### 제1장. AI 협업이 바꾸는 개발 프로세스
- 강화 항목: AI와의 역할 분담(설계/구현/검증), 실패 패턴(환각/과신/컨텍스트 누락), 체크리스트 기반 검증 습관
- **[신규]** 에이전틱 코딩 도구 전경: GitHub Copilot(에이전트 모드), Claude Code(터미널 기반 멀티스텝 리팩토링, Agent Teams), Cursor(Composer 모드), Windsurf(Cascade), Gemini CLI. 도구별 강점과 전략적 조합(IDE 기반 vs 터미널 기반 vs 엔터프라이즈)
- **[신규]** "바이브 코딩(vibe coding)"의 가능성과 한계: 자연어 명세만으로 코드를 생성하는 패러다임이 실무에서 어디까지 통하는가
- 실습: "작은 자동화" 1개를 AI 코딩 도구로 만들고, 사람이 검증해야 하는 지점을 명시한다

#### 제2장. 로컬 개발 환경과 재현 가능한 실행
- 강화 항목: 가상환경, 의존성 고정, 실행 재현성, 크로스 플랫폼 경로
- 실습: `practice/chapter02/code/`에 "실행-검증-산출물 저장" 템플릿 스크립트를 마련한다

### Part 2. MCP와 A2A로 도구·에이전트를 표준화하기

#### 제3장. MCP 개념과 도구/리소스 설계 (2025-11-25 스펙)
- 강화 항목: 도구(tool) 설계 원칙(입출력/에러/제약), 리소스(resource)로 컨텍스트를 다루는 방식
- **[신규]** MCP 2025 스펙 핵심 변경: Tasks(비동기 장기 실행 작업 + 진행률 보고), 구조화된 도구 출력(Structured Tool Outputs), Elicitation(서버→사용자 질의), 서버 발견(.well-known URL)
- **[신규]** MCP 생태계 현황: 17,000+ 커뮤니티 서버, 월 9,700만 SDK 다운로드, MCP Registry(서버 검색/색인 API). OpenAI·Google·Microsoft·Cursor·VS Code 등 주요 플랫폼 채택
- **[신규]** MCP 거버넌스: Anthropic → Linux Foundation AAIF(Agentic AI Foundation) 이관. Anthropic, Block, OpenAI 공동 설립
- 실습: 최소 MCP 서버(로컬 STDIO) 설계 문서 + 스켈레톤 코드. Tasks primitive를 사용한 비동기 작업 1개 포함

#### 제4장. 첫 MCP 서버 만들기: 외부 API 래핑과 A2A 개요
- 강화 항목: 인증키 처리(.env), 실패 처리(타임아웃/재시도), 로깅, 테스트 가능한 구조
- **[신규]** OAuth 2.1 인증: 원격 MCP 서버의 표준 인증 프레임워크
- **[신규]** A2A(Agent-to-Agent) 프로토콜 개요: MCP가 "에이전트↔도구"라면 A2A는 "에이전트↔에이전트" 통신 표준. Agent Card(능력 광고), Client/Remote 모델, 멀티모달 지원(텍스트/오디오/비디오), gRPC(v0.3). Google 주도, Linux Foundation 이관, 50+ 기업 참여
- **[신규]** 프로토콜 선택 기준: MCP vs A2A vs 직접 API 호출 — 언제 어떤 표준을 선택하는가
- 실습: 외부 API 1개를 OAuth 인증이 포함된 MCP 서버로 래핑하고, 실제 호출 결과를 저장한다

### Part 3. 에이전트 프레임워크로 워크플로우를 만든다

#### 제5장. LangChain과 도구 호출의 진화: "도구를 쓰는 에이전트"
- 강화 항목: 툴 선택 실패를 줄이는 프롬프트 구조, 관측 가능성(로그/트레이스), 입력 검증
- **[신규]** 구조화된 출력(Structured Outputs): JSON 스키마 보장 출력, 도구 호출의 신뢰성 향상
- **[신규]** 고급 도구 호출 패턴: Tool Search(30+ 도구 라이브러리에서 컨텍스트 85% 절약), 프로그래밍 방식 도구 호출(Programmatic Tool Calling)
- **[신규]** OpenAI Agents SDK 소개: agent, handoff, guardrails, tracing 4대 요소. Responses API 기반, 100+ LLM 지원(프로바이더 무관). Assistants API 종료(2026.8) 이후의 표준
- 실습: MCP 도구 2개 이상을 LangChain과 OpenAI Agents SDK 각각으로 연결해 "입력 → 처리 → 파일 저장" 파이프라인을 만들고 차이를 비교한다

#### 제6장. LangGraph 1.0으로 상태·분기·반복을 제어한다
- 강화 항목: 상태 모델링, 조건 분기, 반복(재시도/검증 루프), 실패 복구
- **[신규]** LangGraph 1.0(2025.10): 첫 안정 릴리스, v2.0까지 API 안정성 보장. 월 617만 다운로드
- **[신규]** LangSmith 통합: 멀티스텝 워크플로우의 중첩 스팬(nested span) 트레이싱, 평가 워크플로우
- **[신규]** LangGraph + 장기 메모리: MongoDB/Redis 연동을 통한 대화 간 상태 유지
- 실습: "초안 생성 → 검증 → 수정" 순환 워크플로우를 LangGraph 1.0으로 구현하고, LangSmith로 실행 트레이스를 확인한다

#### 제7장. 멀티에이전트 시스템: 프레임워크 비교와 A2A 실전
- 강화 항목: 언제 멀티에이전트가 필요한가, 프레임워크 선택 기준, Human-in-the-loop 지점
- **[신규]** 프레임워크 비교(2026 기준): LangGraph(상태 기반 오케스트레이션) vs OpenAI Agents SDK(handoff 기반) vs Google ADK(Sequential/Parallel/Loop Agent, 계층적 멀티에이전트) vs CrewAI(역할 기반) vs AutoGen v0.4/AG2(비동기 이벤트 메시징, Magentic-One)
- **[신규]** A2A 실전: Agent Card 작성, Client/Remote 에이전트 간 태스크 위임, 이종 프레임워크 에이전트 연동
- **[신규]** 로우코드/노코드 대안: n8n(15만+ GitHub stars), Dify(11.4만+ stars) — 언제 코드 기반 대신 시각적 도구가 적합한가
- 실습: 동일 과제를 (1) LangGraph (2) OpenAI Agents SDK (3) Google ADK 중 2개로 구현해 아키텍처·코드량·디버깅 용이성을 비교한다

### Part 4. 지식 기반 시스템: RAG와 에이전트 메모리

#### 제8장. RAG의 기본과 에이전트 메모리 아키텍처
- 강화 항목: 검색 품질, 인용/출처 추적, 재현 가능한 평가, 캐싱과 비용
- **[신규]** 에이전트 메모리 분류 체계: Working Memory(현재 컨텍스트 윈도우), Episodic Memory(과거 이벤트·결과 저장), Semantic Memory(일반화된 지식), Procedural Memory(학습된 도구 사용 패턴)
- **[신규]** 메모리 구현 전략: 벡터 DB + 시간 메타데이터(Episodic), 지식 그래프(Semantic), 파인튜닝/프롬프트 패턴(Procedural). LangGraph + MongoDB/Redis 조합
- **[신규]** RAG vs 메모리: RAG는 외부 지식 검색, 메모리는 에이전트 경험 축적 — 상호보완 관계
- 실습: 작은 문서 코퍼스를 대상으로 "검색 → 답변 → 출처 표시" 흐름을 만들고, Episodic Memory를 추가해 이전 질의 결과를 활용하는 개선판을 구현한다

#### 제9장. GraphRAG/LightRAG/KAG: 관계 기반 추론으로 확장
- 강화 항목: 그래프 기반 접근의 장단점, 데이터 준비, 평가 설계
- 실습: 동일 질의 세트를 Vector RAG vs Graph 기반 방식으로 비교하고, 실제 결과를 저장한다

### Part 5. 신뢰성·보안·운영·배포

#### 제10장. 에이전트 보안과 신뢰성: 검증에서 가드레일까지
- 강화 항목: 검증 전략(교차검증/출처 일치), 실패 사례 수집, 품질 게이트
- **[신규]** OWASP AI Agent Security Top 10 (2026): 프롬프트 인젝션, 도구 오용, 데이터 유출 등 에이전트 고유 보안 위협 분류
- **[신규]** 다층 가드레일 아키텍처: 입력 가드레일(PII 제거, 탈옥 탐지) → 실행 샌드박싱(gVisor/Firecracker microVM, 네트워크 이그레스 허용목록) → 출력 가드레일(정책 규칙 검사) → ID/접근 제어(에이전트별 최소 권한 IAM)
- **[신규]** 프레임워크 내장 가드레일: OpenAI Agents SDK guardrails primitive, Google ADK safety 설정
- **[신규]** 에이전트 평가 프레임워크: GAIA(멀티스텝 계획), WebArena(웹 상호작용), Terminal-Bench(CLI 워크플로우), CLEAR(Cost/Latency/Efficacy/Assurance/Reliability), LLM-as-Judge(조직 59.8% 사용)
- 실습: "검증 실패 시 재시도/사람 승인"이 포함된 워크플로우를 구성하고, 입력/출력 가드레일을 적용한다

#### 제11장. Human-in-the-loop 워크플로우 설계
- 강화 항목: 승인 요청 UX, 로그/감사, 고위험 작업 분리
- **[신규]** 점진적 배포 전략: Shadow Mode(분석만, 실행 안 함) → 샌드박스 계정 → 스테이징 → 소규모 프로덕션 → 전체 배포
- **[신규]** 감사 로그 설계: 암호화 서명, 변조 방지, 에이전트 행동의 완전한 추적 가능성
- 실습: 승인 없이는 실행되지 않는 단계(예: 외부 시스템 변경)를 포함한 플로우를 만들고, Shadow Mode 단계를 추가한다

#### 제12장. 배포·관측·비용 최적화
- 강화 항목: 운영 메트릭, 비용 추정, 키 관리, 최소 권한
- **[신규]** 관측성 플랫폼 비교: LangSmith(LangChain 네이티브, 무료 5k 트레이스/월) vs Arize Phoenix(엔터프라이즈) vs Langfuse(오픈소스, 셀프호스팅) vs Braintrust(평가 중심) vs Helicone(비용 추적 특화)
- **[신규]** 비용 최적화를 설계 시점부터: 모델 라우팅(단순 작업→저가 모델, 복잡 추론→고성능 모델), 적응형 추론 제어(effort level), 토큰 효율적 도구 호출(Tool Search로 191,300 토큰 절약 사례)
- **[신규]** MCP Registry와 서버 발견: 프로덕션 환경에서 MCP 서버 관리·배포·모니터링
- 실습: 로컬/컨테이너 중 한 가지 방식으로 실행을 재현하고, 관측성 도구(LangSmith 또는 Langfuse)를 연동해 운영 체크리스트를 작성한다

---

## 4) 장별 "필수 포함 요소" 체크리스트

각 `docs/ch{N}.md`는 아래를 최소 1회 이상 포함한다.

- 학습목표(2–4개)와 선수지식 명시
- 의사결정 포인트(무엇을 선택해야 하는지)와 트레이드오프 설명
- 실패 사례 1개 이상과 대응(디버깅/재시도/검증)
- 실습 코드 참조 문장: `_전체 코드는 practice/chapter{N}/code/{파일명}.py 참고_`
- 실제 산출물 경로(예: `practice/chapter{N}/data/output/...`)와 해석

---

## 5) 변경 이력

| 날짜 | 버전 | 주요 변경 |
|------|------|----------|
| 2026-01-02 | 1.0 | 초안 작성 (12장 5파트 구성) |
| 2026-02-08 | 2.0 | 2026 기준 갱신: MCP 2025-11-25 스펙, A2A 프로토콜, OpenAI Agents SDK, Google ADK, 에이전트 메모리, OWASP AI Agent Security Top 10, 평가 프레임워크, 관측성 플랫폼, 에이전틱 코딩 도구 전경 반영 |
