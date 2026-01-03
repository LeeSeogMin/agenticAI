<!-- 보관용: 강의계획서 원문 (교재 목차는 repo 루트의 contents.md에 별도 관리) -->
# AI Agent 캡스톤 디자인 강의계획서 (2026학년도)
### AI 코딩 에이전트 협업 중심 · MCP 기반

## 📋 교과목 정보

- **과목명**: AI Agent 캡스톤 디자인
- **학점**: 3학점
- **시간**: 주 3시간 (이론 1.5시간 + 실습 1.5시간)
- **대상**: 대학원생 및 학부 고학년
- **선수과목**: 기본 프로그래밍 이해, 기계학습 기초, LLM 기본 개념
- **필수 사전 준비**: 
  - GitHub Student Developer Pack 등록 (Copilot 무료 사용)
  - VS Code 설치
  - GitHub 계정

---

## 🎯 교과목 개요

### 새로운 개발 패러다임: GitHub Copilot과 함께 코딩하는 시대

2026년 현재, 소프트웨어 개발은 근본적으로 변화했다:

- **GitHub Copilot의 보편화**: 대학생 무료 제공으로 모든 학생이 AI 코딩 도구 사용 가능
- **MCP 산업 표준화**: Model Context Protocol이 Linux Foundation으로 이관되어 10,000+ 서버, 97M+ SDK 다운로드 달성
- **Agent 기반 개발**: 86%의 기업이 에이전트 기반 시스템에 투자 (총 72억 달러)
- **코드 작성의 진화**: 직접 타이핑에서 AI와의 대화를 통한 설계·검증·생성으로

### 강좌의 핵심 철학: GitHub Copilot 무료 활용

**본 강좌는 전통적인 코딩 교육이 아닙니다.** 학생들은:

1. **GitHub Copilot (무료)**과 협업하여 코드를 작성합니다
   - 모든 학생이 GitHub Student Developer Pack으로 무료 사용
   - VS Code에서 실시간 코드 생성 및 Chat 활용
   - Copilot Workspace로 전체 프로젝트 설계
2. **MCP를 기본 인프라**로 활용하여 모든 시스템을 구축합니다
3. **설계·아키텍처·검증**에 집중하고, 구현은 Copilot에게 위임합니다
4. **Copilot이 생성한 코드를 이해하고 개선**하는 역량을 배양합니다

### 실습 환경의 전제

- **필수**: GitHub Copilot (대학생 무료)
- **필수**: Claude Desktop + MCP (메인 개발 환경)
- **선택**: Cursor, Windsurf (유료지만 더 강력한 기능 원하는 학생)
- 모든 Agent는 **MCP 프로토콜 기반**으로 외부 시스템과 통합
- GitHub Copilot과의 대화(프롬프트)가 코드 작성보다 중요한 스킬

본 강좌는 AI 시대의 개발자가 갖춰야 할 **Agent 설계 역량, MCP 통합 능력, GitHub Copilot 협업 스킬**을 배양합니다.

---

## 🎓 교육 목표

### 핵심 역량 (AI 코딩 에이전트 협업 중심)

1. **AI 협업 개발 역량**
   - GitHub Copilot/Cursor/Windsurf와 효과적으로 협업하는 프롬프팅 능력
   - AI가 생성한 코드를 이해하고 검증·개선하는 역량
   - 복잡한 시스템을 AI와 함께 설계하고 구현하는 능력

2. **MCP 마스터리 (필수)**
   - MCP 서버 설계 및 구축 (모든 프로젝트의 기본)
   - MCP를 통한 도구/데이터/워크플로우 통합
   - MCP 생태계 활용 및 커스터마이징
   - Code Execution with MCP를 통한 효율적 Agent 구현

3. **Agent 아키텍처 설계**
   - LangGraph, CrewAI, AutoGen의 적절한 선택과 활용
   - 멀티에이전트 오케스트레이션 설계
   - 상태 관리 및 워크플로우 최적화
   - Human-in-the-loop 통합 설계

4. **지식 기반 시스템 구축**
   - GraphRAG, KAG, LightRAG를 활용한 신뢰성 있는 검색
   - Knowledge Graph 기반 추론 시스템
   - 환각 방지 및 출처 검증 메커니즘

5. **프로덕션 배포 실무**
   - Docker/Kubernetes 기반 배포
   - 모니터링 및 observability
   - 비용 최적화 및 성능 튜닝

### 학습 성과

**본 강좌 이수 후 학생들은:**

- AI 코딩 에이전트를 활용하여 복잡한 Agent 시스템을 **10배 빠르게** 구축할 수 있다
- MCP 기반으로 모든 외부 시스템(DB, API, 파일, 클라우드)을 Agent와 **표준화된 방식**으로 통합할 수 있다
- 비즈니스 요구사항을 Agent 워크플로우로 설계하고, AI와 협업하여 **프로덕션 레벨**로 구현할 수 있다
- AI가 생성한 코드의 품질을 평가하고, **리팩토링 및 최적화**할 수 있다
- 신뢰성 있는 Agent 시스템을 구축하기 위한 **검증·모니터링·개선** 파이프라인을 설계할 수 있다

---

## 📚 주차별 강의 계획

### **Part 1: GitHub Copilot 기반 AI 협업 개발 환경 구축 (1-3주)**

#### **1주차: GitHub Copilot과 함께하는 개발 - 무료로 시작하기**

**이론**
- **GitHub Copilot이 무료인 이유와 활용법**
  - GitHub Student Developer Pack 등록 방법
  - 대학 이메일 인증으로 Copilot Pro 무료 사용
  - VS Code에서 Copilot 설정
- **GitHub Copilot의 3가지 핵심 기능**
  - **Code Completion**: 실시간 코드 자동완성
  - **Copilot Chat**: 자연어로 코드 요청 및 설명
  - **Copilot Workspace**: 전체 프로젝트 이해 및 생성
- **효과적인 Copilot 활용법**
  - 좋은 주석 작성으로 정확한 코드 생성
  - Chat에서 명확한 요구사항 전달
  - Copilot 제안 수락/거부/수정 전략
  - Tab vs Enter: 언제 무엇을 누를까?

**실습** (모든 실습은 GitHub Copilot과 함께)
- **환경 설정 (중요!)**
  1. GitHub Student Developer Pack 신청
  2. VS Code 설치 및 GitHub Copilot Extension 설치
  3. Copilot Chat 활성화
  4. Python 환경 구축 (Copilot에게 요청)
  
- **첫 Copilot 협업 경험**
  ```python
  # Copilot에게 주석으로 요청
  # Create a web scraper that fetches news articles from BBC
  # Use requests and BeautifulSoup
  # Extract title, date, and summary
  ```
  - Copilot이 생성한 코드 확인
  - Chat으로 "에러 처리 추가해줘" 요청
  - 실행 및 테스트

- **Copilot Chat 실습**
  - `/explain` 명령어로 코드 설명 받기
  - `/fix` 명령어로 에러 수정
  - `/tests` 명령어로 테스트 케이스 생성
  - 자연어로 리팩토링 요청

**과제**: 
- GitHub Copilot을 활용하여 간단한 데이터 파이프라인 구축
- 제출물: 코드 + Copilot Chat 스크린샷 + 회고 (300자)
- 회고 내용: "Copilot이 도움된 부분, 직접 수정한 부분, 배운 점"

---

#### **2주차: MCP 필수 기초 - GitHub Copilot으로 서버 개발하기**

**이론**
- **MCP가 필수인 이유**
  - Agent와 외부 세계를 연결하는 유일한 표준
  - Linux Foundation 기부로 산업 표준 확립
  - 10,000+ 서버, 97M+ SDK 다운로드
- **MCP 핵심 개념**
  - Client-Server 아키텍처
  - Resources (데이터), Tools (기능), Prompts (템플릿)
  - Transport: STDIO (로컬), SSE (원격)
- **MCP 생태계**
  - 주요 공식 서버: Filesystem, GitHub, Google Drive, Slack, PostgreSQL, Puppeteer
  - 커뮤니티 서버: 1000+ 통합
  - Docker Hub MCP 네임스페이스

**실습** (GitHub Copilot 적극 활용)

**[실습 1] Claude Desktop + MCP 통합**
1. Claude Desktop MCP 설정 파일 작성
   - VS Code에서 `claude_desktop_config.json` 생성
   - Copilot Chat에 요청: "Claude Desktop MCP config for filesystem server"
   - Copilot이 생성한 JSON 확인 및 수정
   
2. 공식 MCP 서버 연결
   - Copilot에게 주석으로 요청:
   ```python
   # Install and configure MCP servers:
   # - filesystem (for file operations)
   # - fetch (for web requests)
   # - github (for repository access)
   ```
   - Copilot이 생성한 설치 스크립트 실행

**[실습 2] 첫 MCP 서버 개발 (GitHub Copilot과 협업)**
```python
# GitHub Copilot에게 주석으로 요청:
# Create an MCP server in Python that wraps OpenWeatherMap API
# Requirements:
# - Use STDIO transport
# - Implement get_current_weather(city) tool
# - Implement get_forecast(city, days) tool
# - Add proper error handling
# - Type hints required
```

**단계별 Copilot 활용:**
1. **구조 생성**: 위 주석 작성 → Tab 눌러서 Copilot 제안 수락
2. **검토**: 생성된 코드 읽고 이해하기
3. **개선**: Copilot Chat에 "에러 처리 강화해줘", "로깅 추가해줘" 요청
4. **테스트**: Copilot Chat에 "/tests 테스트 코드 작성해줘" 요청
5. **Claude Desktop 연동**: 생성한 MCP 서버를 Claude와 연결

**[실습 3] MCP 서버 패키징**
- Copilot에게 요청:
```python
# Create a Dockerfile for this MCP server
# Use Python 3.11 slim
# Install dependencies
# Set up STDIO entrypoint
```
- Docker 이미지 빌드 및 실행
- Copilot Chat으로 Dockerfile 최적화

**실습 팁:**
- Copilot이 전체 코드를 한 번에 생성하지 못하면, 함수별로 나눠서 요청
- 생성된 코드는 반드시 실행해보고 에러 발생 시 Copilot Chat의 `/fix` 활용
- 이해 안 되는 부분은 Copilot Chat의 `/explain` 활용

**과제**: 
GitHub Copilot을 활용하여 특정 API(뉴스/주식/날씨 등)를 MCP 서버로 래핑
- 제출물:
  1. MCP 서버 코드 (GitHub 저장소)
  2. Claude Desktop에서 작동하는 스크린샷
  3. Copilot 협업 로그 (주요 프롬프트 5개 + 수정한 부분 설명)
  4. README.md (Copilot에게 작성 요청)

---

#### **3주차: Agent 기본 - LangChain + MCP 통합**

**이론**
- **Agent의 구성 요소 재정의 (MCP 시대)**
  - LLM (두뇌)
  - MCP Tools (손과 발)
  - Memory (기억)
  - Planning (계획)
- **LangChain과 MCP의 만남**
  - `langchain-mcp` 패키지
  - MCP Tools를 LangChain Agent에 통합
  - 기존 LangChain Tools vs MCP Tools
- **Agent 추론 패턴**
  - ReAct (Reasoning + Acting)
  - Function Calling
  - Tool Selection 전략

**실습** (AI 코딩 에이전트 적극 활용)
- **MCP 기반 LangChain Agent 구축**
  - AI에게 "LangChain Agent에 MCP filesystem 서버 연결" 요청
  - AI가 생성한 코드 실행 및 디버깅
  - 여러 MCP 서버를 하나의 Agent에 통합
- **실용 Agent 개발**
  - "GitHub 이슈를 읽고 분석하여 Slack에 보고하는 Agent"
  - 필요한 MCP 서버: github, slack
  - AI와 대화하며 워크플로우 설계 → 구현 → 테스트
- **ReAct 패턴 구현**
  - AI에게 "웹 검색 + 계산 + 파일 저장이 가능한 ReAct Agent" 요청
  - 생성된 코드의 사고 과정(reasoning) 분석

**과제**: 3개 이상의 MCP 서버를 활용하는 실용 Agent 구축 (예: 이메일 읽기 → 문서 분석 → 응답 초안 작성 → 승인 요청)

---

### **Part 2: 프레임워크 마스터리 (4-7주)**
*모든 실습은 AI 코딩 에이전트 + MCP 기반*

#### **4주차: LangGraph - 복잡한 상태 기계를 AI와 함께 구축**

**이론**
- **LangGraph가 필요한 이유**: 순환(cycle), 조건 분기, 상태 관리
- **핵심 개념**
  - StateGraph: 상태 중심 설계
  - Nodes: 각 단계의 함수
  - Edges: 조건부 라우팅
  - Checkpointing: 대화 이력 저장
- **MCP와의 통합**
  - LangGraph Node에서 MCP Tools 호출
  - 여러 MCP 서버를 오케스트레이션

**실습** (AI 코딩 에이전트 적극 활용)
- **AI에게 LangGraph 구조 설계 요청**
  - "고객 지원 워크플로우를 LangGraph로 설계해줘: 문의 분류 → 정보 검색 → 응답 생성 → 검증"
  - AI가 생성한 그래프 구조 검토 및 개선
- **MCP 기반 순환 워크플로우**
  - "코드 리뷰 Agent를 만들어줘: 코드 읽기(MCP filesystem) → 리뷰 → 수정 제안 → 재검증 (순환)"
  - AI와 대화하며 순환 조건 정의
- **LangGraph Studio 활용**
  - 시각적 디버깅
  - 각 노드의 상태 추적
  - AI에게 "디버깅 포인트 추가" 요청

**과제**: AI와 협업하여 복잡한 승인 프로세스를 LangGraph + MCP로 구현 (최소 5개 노드, 2개 이상 순환)

---

#### **5주차: 멀티에이전트 프레임워크 개요 - CrewAI, AutoGen**

**이론**
- **멀티에이전트 시스템의 필요성**
  - 단일 Agent의 한계
  - 역할 분담과 협업의 효율성
  - 복잡한 태스크 분해

- **CrewAI 개념** (간단 소개)
  - Role-based 설계: Agent마다 명확한 역할
  - Crew 구성: Agents + Tasks + Process
  - 사용 사례: 콘텐츠 제작, 리서치, 데이터 분석
  - 장점: 빠른 프로덕션, 명확한 구조
  - 단점: 복잡한 순환 로직 구현 어려움

- **AutoGen 개념** (간단 소개)
  - Conversation-driven 멀티에이전트
  - ConversableAgent, GroupChat
  - 사용 사례: 코드 생성, 논문 작성, 문제 해결
  - 장점: 유연한 협업, Human-in-the-loop 용이
  - 단점: 상태 관리 복잡

- **프레임워크 비교 정리**
  - LangGraph: 복잡한 상태 관리, 순환 워크플로우
  - CrewAI: 역할 기반 협업, 빠른 개발
  - AutoGen: 대화 중심, 유연한 협업

**실습** (GitHub Copilot 활용, 간단한 예제만)

**[간단한 CrewAI 체험]**
```python
# GitHub Copilot에게 요청:
# Create a simple CrewAI crew with 2 agents:
# - Researcher: searches web for AI news
# - Writer: summarizes the findings
# Keep it minimal, use sequential process
```
- Copilot이 생성한 코드 실행
- 역할 기반 협업 개념 이해

**[간단한 AutoGen 체험]**
```python
# GitHub Copilot에게 요청:
# Create a simple AutoGen conversation:
# - AssistantAgent: code generator
# - UserProxyAgent: code executor
# Generate a function to calculate fibonacci
```
- Agent 간 대화 패턴 관찰

**실습 포인트:**
- 이번 주는 **개념 이해**가 목표
- 실제 프로젝트에서는 **LangGraph 중심** 사용 권장
- CrewAI/AutoGen은 특정 상황에서 **보조적 활용**

**과제**: 
- LangGraph, CrewAI, AutoGen 비교표 작성 (각 프레임워크의 장단점, 적합한 사용 사례)
- 자신의 캡스톤 프로젝트에 어떤 프레임워크가 적합한지 선택 및 근거 작성 (300자)

---

#### **6주차: Code Execution with MCP - 차세대 Agent 패턴**

**이론**
- **전통적 Tool Calling의 한계**
  - 모든 도구 정의를 컨텍스트에 로드
  - 중간 결과가 토큰 낭비
  - 대용량 데이터 처리 불가
- **Code Execution with MCP**
  - Tools를 코드로 호출
  - On-demand 로딩
  - 데이터 필터링 before LLM
- **Cloudflare의 "Code Mode" 사례**

**실습** (AI 코딩 에이전트 필수)
- **기존 Tool Calling 방식 구현**
  - AI에게 "Google Drive 문서를 읽어서 Salesforce에 저장하는 Agent" 요청
  - 토큰 사용량 측정
- **Code Execution 방식으로 전환**
  - AI에게 "위 코드를 Code Execution with MCP 패턴으로 리팩토링해줘"
  - 토큰 절감 효과 비교
- **대용량 데이터 처리**
  - 10MB 파일 처리를 두 방식으로 비교
  - AI와 함께 병목 지점 분석

**과제**: AI와 협업하여 Code Execution with MCP를 활용한 ETL 파이프라인 구축

---

#### **7주차: 프레임워크 선택 전략 및 프로젝트 설계**

**이론**
- **프레임워크 선택 의사결정**
  - LangGraph: 복잡한 상태 관리, 명확한 흐름 제어, 순환 워크플로우 필요 시
  - CrewAI: 역할 기반 협업, 빠른 프로덕션, 간단한 파이프라인
  - AutoGen: 유연한 대화, Human-in-the-loop 중심, 연구/프로토타입
  - Code Execution with MCP: 대용량 데이터, 토큰 효율성 중요 시

- **실전 선택 기준**
  - 워크플로우 복잡도 (순환, 조건 분기)
  - 상태 관리 필요성
  - 개발 속도 vs 유연성
  - 팀 구조 vs 순차 처리

- **Hybrid 접근**
  - LangGraph (오케스트레이션) + MCP (도구)
  - LangGraph (복잡 로직) + CrewAI (단순 태스크)

**실습** (GitHub Copilot과 비교 분석)

**[Case Study 분석]**
- **케이스 1**: 고객 지원 자동화
  - Copilot에게 요청: "LangGraph와 CrewAI 중 어떤 게 적합한지 비교해줘"
  - 요구사항 분석, 프레임워크 선택 근거
  
- **케이스 2**: 학술 논문 자동 리뷰
  - Copilot Chat에서 의사결정 트리 작성
  - 각 프레임워크별 구현 복잡도 추정

**[프로젝트 설계 실습]**
- 캡스톤 프로젝트 아이디어 구체화
- Copilot과 대화하며 아키텍처 설계
- 필요한 MCP 서버 식별
- 프레임워크 선택 및 근거 작성

**캡스톤 프로젝트 제안서 제출 (중간 평가)**
- 프로젝트 개요 및 목표
- 선택한 프레임워크 및 이유
- 필요한 MCP 서버 목록
- 개발 일정 (8-16주차)
- 기대 효과 및 혁신성

---

### **Part 3: 지식 기반 Agent 시스템 (8-10주)**
*RAG 진화에서 Domain-specific Agent Marketplaces까지*

#### **8주차: GraphRAG & KAG - 지식 그래프 기반 검색**

**이론**
- **전통적 RAG의 한계**
  - Vector 검색만으로는 관계성 파악 어려움
  - Multi-hop 추론 실패
  - 맥락 손실 및 토큰 낭비
  
- **GraphRAG (Microsoft)**
  - Entity Extraction: 문서에서 개체 추출
  - Community Detection: 관련 개체 그룹화
  - Hierarchical Summarization: 계층적 요약
  - 90% 환각 감소, Sub-50ms 쿼리 속도
  
- **LightRAG (HKUDS)**
  - GraphRAG의 경량화 버전
  - 10배 토큰 절감
  - Dual-level retrieval: Local + Global
  
- **KAG (Knowledge Augmented Generation)**
  - Structured knowledge 통합
  - Multi-hop reasoning 지원
  - Domain-specific knowledge graph
  - OpenSPG 엔진 기반

**실습** (GitHub Copilot 활용)

**[실습 1] GraphRAG 파이프라인 구축**
```python
# Copilot에게 요청:
# Implement Microsoft GraphRAG pipeline:
# 1. Document ingestion
# 2. Entity extraction using LLM
# 3. Build knowledge graph with Neo4j
# 4. Community detection
# 5. Query with hierarchical search
```
- Copilot이 생성한 코드 실행
- 자신의 문서 코퍼스로 테스트

**[실습 2] LightRAG로 효율화**
```python
# Copilot Chat에:
"위 GraphRAG 코드를 LightRAG 패턴으로 최적화해줘.
토큰 사용량 줄이고 dual-level retrieval 구현"
```

**[실습 3] KAG 시스템 구축**
```python
# Copilot에게 요청:
# Build KAG system for legal domain:
# - Extract entities: laws, cases, precedents
# - Build domain knowledge graph
# - Implement multi-hop reasoning
# - Connect to LangGraph agent
```

**[실습 4] MCP 통합**
- GraphRAG를 MCP 서버로 래핑
- LangGraph Agent에서 GraphRAG MCP 서버 호출
- Copilot으로 통합 코드 생성

**과제**: 
- 특정 도메인(의료/법률/금융) 문서 100개로 GraphRAG 시스템 구축
- Vector RAG vs GraphRAG 성능 비교 (정확도, 속도, 토큰 사용량)
- GitHub Copilot 활용 로그 제출

---

#### **9주차: DeepAgents - 심층 강화학습 Agent**

**이론**
- **DeepAgents란?**
  - Deep Learning + Agentic Workflows
  - 강화학습 기반 의사결정
  - Self-improving Agents
  - 복잡한 환경에서 자율 학습

- **주요 개념**
  - Reward Modeling: Agent 행동 평가
  - Policy Learning: 최적 전략 학습
  - Exploration vs Exploitation
  - Multi-step Planning

- **실제 응용 사례**
  - 게임 AI (AlphaGo 스타일)
  - 로봇 제어
  - 트레이딩 Agent
  - 자율 연구 Agent

- **LLM과의 결합**
  - LLM이 policy를 언어로 표현
  - Reward signal로 LLM 미세조정
  - Chain-of-Thought + RL

**실습** (GitHub Copilot 활용)

**[실습 1] 간단한 RL Agent**
```python
# Copilot에게 요청:
# Create a simple reinforcement learning agent:
# - Environment: text-based game
# - Agent: LLM + reward model
# - Training loop with experience replay
# - Policy improvement over iterations
```

**[실습 2] Self-improving Code Agent**
```python
# Copilot Chat에:
"DeepAgents 패턴으로 코드 생성 Agent 만들어줘:
1. LLM이 코드 생성
2. 테스트 실행으로 reward 계산
3. 실패하면 반성하고 재시도
4. 성공 패턴 학습"
```

**[실습 3] Research Agent**
- 논문 검색 → 읽기 → 요약 → 평가
- Reward: 요약 품질 점수
- 여러 반복으로 요약 능력 개선
- Copilot과 함께 구현

**토론**: 
- DeepAgents의 윤리적 문제 (자율성 vs 통제)
- Self-improving Agent의 위험성
- 실무 적용 가능성

**과제**: 
- 간단한 게임 환경에서 RL Agent 구현
- 10회 iteration 후 성능 개선 입증
- Reward 함수 설계 및 정당화

---

#### **10주차: Advanced RAG + Domain-specific Agent Marketplaces**

**이론**

**Part A: Advanced RAG 기법**
- **Hybrid Search**
  - Vector (semantic) + Keyword (lexical)
  - Graph (relational) 통합
  - Re-ranking 전략
  
- **Agentic RAG**
  - Agent가 검색 전략 결정
  - 자율적으로 쿼리 리라이팅
  - Multi-step retrieval with reflection
  
- **Production RAG Best Practices**
  - Chunking 전략 (고정 vs 의미 기반)
  - Metadata filtering
  - Citation & source tracking
  - Cache 최적화

**Part B: Domain-specific Agent Marketplaces**
- **Agent Marketplace 개념**
  - 전문 도메인별 Agent 패키징
  - Reusable Agent Components
  - Agent-as-a-Service
  
- **도메인 특화 Agent 예시**
  - Legal Agent: 계약서 검토, 판례 분석
  - Medical Agent: 진단 보조, 문헌 조사
  - Financial Agent: 리스크 분석, 포트폴리오 최적화
  - Academic Agent: 논문 리뷰, 연구 동향 분석
  
- **Marketplace 아키텍처**
  - Agent Discovery & Registry
  - MCP 기반 표준 인터페이스
  - Quality Assurance & Rating
  - Usage-based Pricing

**실습** (GitHub Copilot 활용)

**[실습 1] Hybrid RAG 시스템**
```python
# Copilot에게 요청:
# Build hybrid RAG system:
# - Vector search with ChromaDB
# - Graph search with Neo4j  
# - Keyword search with BM25
# - Fusion re-ranking
# - LangGraph agent orchestration
```

**[실습 2] Agentic RAG**
```python
# Copilot Chat에:
"Self-reflective RAG agent 만들어줘:
1. 쿼리 분석
2. 검색 전략 결정 (vector/graph/keyword)
3. 검색 수행
4. 결과 평가 (충분한가?)
5. 불충분하면 쿼리 리라이팅 후 재검색
LangGraph로 구현"
```

**[실습 3] Domain Agent 패키징**
- 의료/법률/금융 중 1개 도메인 선택
- 해당 도메인의 Agent 구축:
  - Domain knowledge graph
  - Specialized prompts
  - Domain-specific tools (MCP)
  - Evaluation metrics
- MCP 서버로 패키징
- README에 사용법 문서화

**[실습 4] Mini Agent Marketplace**
```python
# Copilot에게 요청:
# Create agent marketplace system:
# - Agent registry (JSON/DB)
# - Discovery API (search by domain/capability)
# - Agent metadata (description, pricing, rating)
# - Integration example with LangGraph
```

**그룹 활동**:
- 각 팀이 도메인 Agent 1개 개발
- 서로의 Agent를 Marketplace에 등록
- Cross-team Agent 활용 (A팀 Agent → B팀 프로젝트)
- 평가 및 피드백

**과제**:
- Domain-specific Agent 1개 완성 (MCP 서버)
- Agent 카탈로그 작성 (기능, 입출력, 제약사항)
- 다른 팀 Agent 1개를 자신의 시스템에 통합
- Marketplace 개념이 실무에 어떻게 적용될지 전망 (500자)

---

### **Part 4: 프로덕션 및 고급 주제 (11-13주)**

#### **11주차: 멀티에이전트 시스템 고급 설계**

**이론**
- Multi-agent Orchestration 패턴
  - Supervisor Pattern
  - Peer-to-peer Collaboration
  - Hierarchical Teams
- Agent Communication Protocols
- Conflict Resolution
- Resource Allocation
- Scalability 고려사항

**실습**
- Supervisor Agent를 활용한 팀 관리
- 병렬 작업 처리 및 결과 통합
- Agent 간 메시지 큐 구현
- Load Balancing 전략

**과제**: 복잡한 비즈니스 프로세스 자동화를 위한 멀티에이전트 아키텍처 설계

---

#### **12주차: 신뢰성 보장 + Human-in-the-Loop 통합**

**이론**

**Part A: 환각 방지 및 신뢰성 보장**
- **Hallucination의 원인과 유형**
  - 지식 부족(Knowledge Gap)
  - 과도한 일반화
  - Context 혼동
  
- **검증 전략**
  - Source Citation: 모든 답변에 출처 표시
  - Cross-validation: 여러 소스 비교
  - Confidence Scoring: 확신도 점수화
  - Fact-checking pipeline

- **Guardrails 구현**
  - Input Validation: 악의적 입력 차단
  - Output Verification: 생성 내용 검증
  - Safety Constraints: 안전 규칙 적용

**Part B: Human-in-the-Loop 설계**
- **HITL 필요성**
  - 고위험 의사결정 (의료, 금융, 법률)
  - Agent 학습 및 개선
  - 신뢰 구축
  
- **HITL 패턴**
  - Approval Workflows: 중요 작업 승인
  - Active Learning: 불확실한 경우 질문
  - Feedback Integration: 사용자 피드백 학습
  
- **Explainability (XAI)**
  - Agent가 왜 그런 결정을 했는지 설명
  - Decision path visualization
  - Confidence score 제시

**실습** (GitHub Copilot 활용)

**[실습 1] Guardrails 구현**
```python
# Copilot에게 요청:
# Implement guardrails for LangGraph agent:
# - Input validation (prompt injection detection)
# - Output verification (fact-checking)
# - Confidence scoring (0-1 scale)
# - Source citation (track all sources)
# Use guardrails-ai library
```

**[실습 2] Citation System**
```python
# Copilot Chat에:
"GraphRAG agent에 citation 시스템 추가:
- 모든 답변에 출처 표시
- 출처 신뢰도 점수
- 출처 간 일치도 검증"
```

**[실습 3] Human Approval Workflow**
```python
# Copilot에게 요청:
# Build human-in-the-loop workflow:
# - Agent proposes action
# - Send approval request to human (Slack/Email)
# - Wait for human response
# - Execute if approved, skip if rejected
# - Log all decisions
```

**[실습 4] Monitoring Dashboard**
- Streamlit으로 대시보드 구축
- Agent 활동 실시간 모니터링
- 환각 감지 알림
- Human approval queue
- Copilot Chat: "대시보드에 필요한 모든 위젯 만들어줘"

**[실습 5] LangSmith 트레이싱**
- LangSmith로 Agent 실행 추적
- 각 단계의 입출력 기록
- 에러 발생 지점 식별
- 성능 병목 분석

**프로젝트 중간발표**: 
- 캡스톤 프로젝트 진행상황 및 프로토타입 시연
- 신뢰성 메커니즘 시연
- HITL 워크플로우 시연

---

#### **13주차: 프로덕션 배포 및 운영**

**이론**
- Deployment Architecture
  - Containerization (Docker)
  - Orchestration (Kubernetes)
  - Serverless Options (Cloud Run, Lambda)
- Monitoring & Observability
  - LangSmith, Langfuse
  - Custom Metrics
  - Error Tracking
- Security & Compliance
  - API Key Management
  - Data Privacy
  - Rate Limiting
- Cost Optimization

**실습**
- Docker 컨테이너화 및 배포
- Cloud Run/Azure Container Apps 배포
- Prometheus + Grafana 모니터링
- CI/CD 파이프라인 구축 (GitHub Actions)

**과제**: 프로덕션 체크리스트 작성 및 배포 계획서

---

### **Part 5: 최신 트렌드 및 프로젝트 (14-16주)**

#### **14주차: 2026년 Agent 생태계 전망 및 윤리**

**이론**

**Part A: Agent 생태계 진화**
- **B2A (Business-to-Algorithm)**
  - AI가 vendor를 선택하고 구매 결정
  - 알고리즘 친화적 데이터 구조화
  - Clean data = 경쟁력
  
- **Agent Interoperability**
  - MCP가 가능하게 한 Agent 간 통신
  - Cross-platform Agent 협업
  - Agent marketplace 확산
  
- **Federated Learning for Agents**
  - 데이터 공유 없이 Agent 학습
  - Privacy-preserving collaboration
  - 분산 Agent 네트워크

**Part B: Agent 윤리 및 거버넌스**
- **윤리적 이슈**
  - Agent의 자율성 vs 통제
  - 편향성 (Bias) 문제
  - 투명성 및 설명가능성
  - 책임 소재 (Agent vs 개발자 vs 사용자)
  
- **거버넌스 프레임워크**
  - Agent 행동 규약
  - Safety constraints
  - Audit trail 의무화
  - 인간 최종 결정권
  
- **규제 동향**
  - EU AI Act
  - 한국 AI 기본법
  - 산업별 가이드라인

**특강 및 토론**

**[특강]**
- **산업 전문가 초청 강연** (AI Agent 실무 사례)
  - 금융권 Agent 도입 사례
  - 의료 분야 Agent 활용 및 규제
  - 스타트업의 Agent 기반 비즈니스

**[최신 논문 리뷰 세션]**
- 학생들이 2025-2026년 Agent 관련 주요 논문 발표
- 각 팀 15분 발표
- Topics: GraphRAG, DeepAgents, Agentic RAG 등

**[윤리 토론]**
주제: "Agent가 사람보다 나은 결정을 내린다면?"
- 찬성: Agent의 자율성 확대
- 반대: 인간 통제 유지
- 모더레이터: 교수

**실습**

**[Agent 윤리 체크리스트 작성]**
- 자신의 캡스톤 프로젝트에 대한 윤리 체크리스트
- Copilot과 함께 작성:
  ```
  # Copilot에게:
  "AI Agent 윤리 체크리스트 템플릿 만들어줘:
  - 편향성 평가
  - 투명성 수준
  - 사용자 통제 정도
  - 데이터 프라이버시
  - 오용 가능성"
  ```

**과제**:
- 최신 Agent 논문 1편 리뷰 (5페이지)
- 캡스톤 프로젝트의 윤리적 이슈 분석 (2페이지)
- Agent 생태계 전망 에세이 (500자)

---

#### **15주차: 캡스톤 프로젝트 발표 (1차)**

**프로젝트 최종 발표 및 시연**
- 팀별 20분 발표 + 10분 Q&A
- 실제 동작하는 시스템 데모
- 기술적 도전과 해결 과정 공유
- 동료 평가 및 피드백

**평가 기준**
- 기술적 완성도 (30%)
- 창의성 및 실용성 (25%)
- 시스템 신뢰성 (20%)
- 발표 및 문서화 (15%)
- 팀워크 (10%)

---

#### **16주차: 캡스톤 프로젝트 발표 (2차) 및 총정리**

**프로젝트 최종 발표 (계속)**

**강좌 총정리**
- Agent 기술의 현재와 미래
- 학습 내용 회고
- 지속적 학습 로드맵

**최종 과제 제출**
- 프로젝트 소스 코드 (GitHub)
- 최종 보고서
- 시연 영상
- 배포 문서

---

## 📊 평가 방법

| 항목 | 비중 | 세부 내용 |
|------|------|-----------|
| 출석 및 참여 | 10% | 강의 참석, 토론 참여, AI 활용 패턴 공유 |
| 주간 과제 | 30% | 매주 실습 과제 (10회 × 3%) + **AI 협업 로그 제출** |
| 중간 평가 | 20% | MCP 서버 + Agent 통합 프로젝트 + **AI 활용 보고서** |
| 캡스톤 프로젝트 | 40% | 팀 프로젝트 (최종 발표 + 보고서 + 코드) + **AI 협업 기록** |

### 주간 과제 평가 기준

각 과제는 다음 항목으로 평가:

1. **기능 완성도** (40%)
   - 요구사항 충족도
   - 코드 실행 가능성
   - 에러 처리

2. **AI 협업 역량** (30%) ⭐ 중요
   - AI와의 대화 품질 (프롬프트 명확성)
   - AI 생성 코드 검증 능력
   - 반복 개선 과정
   - **제출 필수**: AI 대화 로그 스크린샷 또는 Cursor/Windsurf 히스토리

3. **MCP 통합** (20%)
   - MCP 서버 활용 또는 개발
   - 표준 프로토콜 준수

4. **문서화** (10%)
   - README 작성
   - 설계 의도 설명

### 캡스톤 프로젝트 평가 기준

| 평가 항목 | 배점 | 세부 기준 |
|----------|------|----------|
| **기술적 완성도** | 30% | MCP 통합, 프레임워크 적절성, 코드 품질 |
| **AI 협업 우수성** | 25% | AI 활용 전략, 생산성 향상, 협업 로그 |
| **창의성 및 실용성** | 20% | 문제 정의, 솔루션 독창성, 실제 활용 가능성 |
| **신뢰성 및 안정성** | 15% | 에러 처리, 검증 메커니즘, 테스트 |
| **발표 및 문서** | 10% | 시연, 기술 문서, 팀워크 |

### AI 협업 로그 제출 가이드

**필수 포함 사항:**
- **GitHub Copilot Chat 스크린샷** (주요 대화 3-5개)
- Copilot 주석 요청 → 생성된 코드 예시
- Copilot이 생성한 코드 중:
  - 그대로 사용한 부분
  - 수정한 부분 (왜 수정했는지)
- `/explain`, `/fix`, `/tests` 명령어 활용 사례
- 전체 개발 시간 중 Copilot이 절약해준 시간 추정

**제출 형식:**
- **VS Code**: Copilot Chat 히스토리 스크린샷 (우클릭 → Export)
- 또는 주요 대화 5개 캡처
- 간단한 리플렉션 (200-300자): "Copilot을 어떻게 활용했고, 무엇을 배웠는가"

**좋은 제출 예시:**
```
[스크린샷 1] 초기 요청: "MCP 서버 구조 생성해줘"
→ Copilot이 생성한 기본 구조

[스크린샷 2] 개선 요청: "/fix 에러 처리 부족"
→ Copilot이 try-except 추가

[스크린샷 3] 테스트: "/tests 이 함수 테스트 케이스"
→ Copilot이 pytest 코드 생성

리플렉션:
"Copilot으로 30분 만에 MCP 서버 프로토타입 완성. 
직접 작성했다면 2-3시간 걸렸을 것. 하지만 에러 처리 
로직은 Copilot이 놓쳤고, 내가 추가함. /fix 명령어가 
특히 유용했음."
```

**나쁜 제출 예시:**
```
[코드만 제출, Copilot 사용 흔적 없음]
또는
"Copilot 사용했어요" (증거 없음)
```

### 우수 사례 인센티브

- **Best AI Collaboration Award**: AI와 가장 효과적으로 협업한 팀/개인
- **Best MCP Integration**: MCP 활용이 우수한 프로젝트
- **Best Production Deployment**: 실제 배포 및 운영 사례

수상팀 혜택:
- 추가 학점 보너스
- 학과 홈페이지 프로젝트 소개
- 산학 협력 프로젝트 우선 추천

---

## 🛠 개발 환경 및 도구

### 필수 도구 (수강 전 준비 필요)

#### AI 코딩 도구 (필수)

**GitHub Copilot (필수 - 대학생 무료)**
- **가격**: GitHub Student Developer Pack으로 무료
- **신청**: [education.github.com/pack](https://education.github.com/pack)
- **기능**:
  - Code Completion: 실시간 코드 자동완성
  - Copilot Chat: VS Code 내 AI 채팅 (`Ctrl+I`)
  - Copilot Workspace: 프로젝트 전체 이해
  - `/explain`, `/fix`, `/tests` 등 명령어
- **설치**: VS Code Extension → "GitHub Copilot" 검색 → 설치

**추가 AI 도구 (선택 - 유료지만 더 강력)**
- **Cursor** (권장): AI-first IDE, Composer 기능 탁월
  - 월 $20 (Pro), 무료 평가판 2주
  - 여러 파일 동시 편집 가능
  - 프로젝트 전체 컨텍스트 이해
  
- **Windsurf** (고급): Agentic IDE
  - Cascade mode: 완전 자율 편집
  - Flow mode: 지속적 컨텍스트 유지
  
- **Claude Code** (CLI 사용자): 터미널 기반
  - 명령줄에서 자율 코딩

**강좌 권장사항:**
- 최소: GitHub Copilot (무료, 모든 학생 필수)
- 추천: GitHub Copilot + Cursor (더 나은 경험)
- 고급: 위 모두 + Claude Code (CLI 전문가)

#### Claude Desktop (필수 - 무료)
- **용도**: MCP 통합 메인 환경
- **다운로드**: [claude.ai/download](https://claude.ai/download)
- **기능**: MCP 서버 연결, Agent 테스트

#### MCP 관련 (필수)
- **MCP 서버 개발**: Python 3.10+, TypeScript/Node.js
- **공식 MCP 서버**: filesystem, fetch, github, postgres, puppeteer
- **MCP SDK**: `@modelcontextprotocol` (TypeScript), `mcp` (Python)
- **Docker**: MCP 서버 패키징 및 배포

#### 개발 도구
- **IDE**: VS Code (Copilot 사용 위해 필수)
- **언어**: Python 3.10+ (주 언어)
- **버전 관리**: Git, GitHub (Student Pack 신청용)
- **컨테이너**: Docker, Docker Compose

#### Agent 프레임워크
- **LangChain 0.3+**: 기본 Agent 구축
- **LangGraph**: 상태 기계 기반 워크플로우
- **CrewAI**: 역할 기반 멀티에이전트
- **AutoGen**: 대화형 협업
- **langchain-mcp**: LangChain ↔ MCP 통합

#### 데이터베이스
- **Vector DB**: ChromaDB, Qdrant
- **Graph DB**: Neo4j, FalkorDB (GraphRAG용)
- **관계형 DB**: PostgreSQL

#### LLM 플랫폼 (필수)
- **Claude API**: Sonnet 4 (주 모델)
- **OpenAI API**: GPT-4 (보조)
- (옵션) **Gemini API**, **Ollama** (로컬)

### 권장 도구

#### 모니터링 & 디버깅
- **LangSmith**: Agent 트레이싱 (필수)
- **LangGraph Studio**: 시각적 디버깅
- **Langfuse**: 오픈소스 observability

#### UI 개발
- **Streamlit**: 빠른 프로토타입
- **Gradio**: ML 모델 UI

#### 클라우드 (프로젝트 배포용)
- **Google Cloud Platform**: Cloud Run, Vertex AI
- **Cloudflare**: MCP 서버 배포
- (옵션) **Azure**, **AWS**

---

### GitHub Copilot 활용 가이드

#### GitHub Student Developer Pack 신청 방법

1. **GitHub 계정 생성** (없다면)
   - [github.com](https://github.com) 가입
   
2. **Student Pack 신청**
   - [education.github.com/pack](https://education.github.com/pack) 방문
   - "Get your pack" 클릭
   - 대학 이메일 (@university.ac.kr) 입력
   - 학생증 또는 재학증명서 업로드
   
3. **승인 대기** (보통 1-3일)
   - 승인되면 이메일 수신
   - GitHub Copilot Pro 무료 활성화

4. **VS Code에서 Copilot 설치**
   - VS Code Extensions → "GitHub Copilot" 검색
   - Install → GitHub 로그인
   - Copilot Chat도 함께 설치

#### GitHub Copilot 사용 패턴

**방법 1: 주석으로 요청 (Code Completion)**
```python
# Create a function that reads CSV file and converts to JSON
# Handle encoding errors and return dict
```
→ Tab 키 눌러서 Copilot 제안 수락

**방법 2: Copilot Chat 활용** (`Ctrl+I`)
```
User: "MCP 서버를 Python으로 만들어줘. 
- GitHub API를 래핑
- 리포지토리 목록, 이슈 목록, 이슈 생성 기능
- STDIO transport 사용
- 에러 핸들링 포함
- 타입 힌트 완비"

Copilot: [전체 코드 생성]
```

**방법 3: 명령어 활용**
- `/explain`: 선택한 코드 설명
- `/fix`: 에러 수정
- `/tests`: 테스트 케이스 생성
- `/doc`: 문서화 주석 추가

#### Copilot과 반복 개선 패턴
1. **초기 요청**: 주석 또는 Chat으로 요구사항 명확히
2. **검토**: Copilot 생성 코드 실행 및 테스트
3. **개선 요청**: Chat에서 "이 부분 리팩토링해줘", "에러 처리 추가해줘"
4. **검증**: 다시 실행, 필요시 3번 반복

#### 코드 리뷰 시 Copilot 활용
```
# 코드 선택 후 Copilot Chat에:
"이 코드의 잠재적 버그 찾아줘"
"성능 개선 포인트 알려줘"
"보안 취약점 체크해줘"
"테스트 케이스 작성해줘"
```

---

## 📖 주요 참고 자료

### 공식 문서
- [LangGraph Documentation](https://langchain-ai.github.io/langgraph/)
- [CrewAI Documentation](https://docs.crewai.com/)
- [AutoGen Documentation](https://microsoft.github.io/autogen/)
- [Model Context Protocol](https://modelcontextprotocol.io/)
- [Microsoft GraphRAG](https://github.com/microsoft/graphrag)

### 논문 및 리서치
- "GraphRAG: Unlocking LLM discovery on narrative private data" (Microsoft Research, 2024)
- "LightRAG: Simple and Fast Retrieval-Augmented Generation" (HKUDS, 2024)
- "KAG: Knowledge Augmented Generation" (OpenSPG, 2025)
- "ReAct: Synergizing Reasoning and Acting in Language Models" (2023)

### 서적
- *Building LLM-Powered Applications* (O'Reilly, 2024)
- *Agentic Graph RAG* (O'Reilly, 2025)
- *Generative AI on AWS* (O'Reilly, 2024)

### 온라인 리소스
- [LangChain Blog](https://blog.langchain.dev/)
- [Anthropic Engineering Blog](https://www.anthropic.com/engineering)
- [AI Agent Hub](https://www.aiagent.app/)
- GitHub Awesome Lists: Awesome-LangChain, Awesome-GraphRAG

---

## 🎯 캡스톤 프로젝트 주제 예시

### 권장 주제 영역

1. **비즈니스 자동화**
   - 고객 지원 자동화 시스템
   - 계약서 검토 및 리스크 분석 Agent
   - 재무 보고서 자동 생성 및 분석

2. **연구 및 지식 작업**
   - 학술 논문 리뷰 자동화
   - 특허 선행기술 조사 Agent
   - 법률/의료 지식 베이스 질의응답

3. **소프트웨어 개발**
   - 코드 리뷰 및 리팩토링 Agent
   - 자동화된 테스트 케이스 생성
   - 기술 문서 자동 생성

4. **데이터 분석**
   - 비정형 데이터 ETL 자동화
   - 대시보드 자동 생성 및 인사이트 추출
   - 실시간 이상 탐지 및 보고

5. **교육 및 학습**
   - 개인화된 학습 경로 추천 Agent
   - 자동 문제 출제 및 채점 시스템
   - 대화형 튜터링 Agent

### 프로젝트 요구사항
- 최소 2개 이상의 Agent 협업
- MCP 또는 외부 도구 통합
- GraphRAG/KAG 등 지식 증강 기술 활용
- Human-in-the-loop 또는 검증 메커니즘
- 배포 가능한 형태의 프로토타입
- 성능 평가 및 개선 사항 문서화

---

## 💡 학습 가이드

### 선수 학습 권장사항
- 기본 프로그래밍 개념 (변수, 함수, 클래스)
- Python 문법 기초 (Copilot이 생성한 코드를 읽을 수 있는 수준)
- Git/GitHub 기본 사용법 (Student Pack 신청 필수)
- REST API 개념
- **GitHub Copilot 사전 경험** (권장, 필수 아님)

### 주간 학습 권장 시간
- 강의 시간: 3시간
- Copilot과 함께하는 실습: 4-6시간
- 과제 및 복습: 3-4시간
- 프로젝트 작업: 4-6시간 (후반부)
- **총 주당 14-19시간**

### GitHub Copilot 활용 전략 ⭐ 중요

#### Level 1: 초보 (1-3주차 목표)
- **주석으로 요청하기**
  ```python
  # Read CSV file and convert to pandas DataFrame
  # Handle missing values by filling with mean
  ```
  → Tab 키로 Copilot 제안 수락
  
- **Copilot Chat 기본 명령어**
  - 코드 선택 → `Ctrl+I` → `/explain` (설명)
  - 에러 발생 → `Ctrl+I` → `/fix` (수정)
  - 함수 선택 → `Ctrl+I` → `/tests` (테스트)
  
- **생성된 코드 그대로 사용**
  - 일단 실행해보고 작동하면 OK
  - 이해 안 되면 `/explain` 활용

#### Level 2: 중급 (4-7주차 목표)
- **복잡한 요구사항 전달**
  ```python
  # Copilot Chat에 요청:
  """
  LangGraph를 사용해서 다음 워크플로우 구현:
  1. 사용자 질문 받기
  2. MCP filesystem으로 관련 문서 검색
  3. 검색 결과 요약
  4. 불충분하면 2번으로 돌아가기 (최대 3회)
  5. 최종 답변 생성
  
  조건: Python 3.10+, 타입 힌트, 에러 처리 포함
  """
  ```
  
- **Copilot 생성 코드 검토 및 개선**
  - 생성된 코드 실행 → 문제 발견 → Chat에 "이 부분 개선해줘" 요청
  - `/fix` 명령어로 버그 수정
  - 여러 번 대화하며 코드 다듬기
  
- **여러 파일 작업**
  - `main.py` 생성 후 → "이를 위한 `config.py` 만들어줘"
  - "이 모듈의 테스트 코드를 `test_main.py`에 만들어줘"

#### Level 3: 고급 (8주차 이후 목표)
- **전체 아키텍처 설계 요청**
  ```
  Copilot Chat:
  "MCP 기반 고객 지원 Agent 시스템 설계해줘.
  - LangGraph로 워크플로우 관리
  - MCP 서버: Slack, Gmail, Database
  - 파일 구조 제안하고 각 파일 코드 생성"
  ```
  
- **성능·보안 분석**
  - 코드 전체 선택 → "성능 병목 찾아줘"
  - "보안 취약점 있는지 체크해줘"
  - "메모리 누수 가능성 분석해줘"
  
- **Copilot과 Pair Programming**
  - Copilot이 초안 생성 → 내가 리뷰 → Chat으로 개선 요청 → 반복
  - 복잡한 로직은 내가 설계 → Copilot이 구현
  - Copilot이 놓친 엣지 케이스를 내가 추가

### 효과적인 GitHub Copilot 협업 패턴

#### 🎯 좋은 주석/프롬프트 작성법

**나쁜 예:**
```python
# Agent 만들어줘
```

**좋은 예:**
```python
# Create a LangGraph agent with the following workflow:
# 1. Receive user question
# 2. Search documents using MCP filesystem server
# 3. Summarize search results
# 4. If insufficient, loop back to step 2 (max 3 times)
# 5. Generate final answer
# Requirements: Python 3.10+, type hints, error handling
```

**더 좋은 예 (Copilot Chat):**
```
"LangGraph StateGraph를 만들어줘:
- Nodes: question_receive, document_search, summarize, answer_generate
- Edges: conditional routing (summarize → document_search if confidence < 0.7)
- State: TypedDict with query, documents, summary, confidence
- 각 node는 async function
- MCP filesystem 서버 통합"
```

#### 🔄 반복 개선 사이클 (GitHub Copilot)

1. **초기 요청**: 주석 또는 Chat으로 명확하게
2. **실행 및 테스트**: Copilot 생성 코드 실행
3. **문제 발견**: 에러, 비효율, 불완전 부분
4. **개선 요청**: 
   - Chat: "에러 처리 강화해줘"
   - Chat: "이 함수 최적화해줘"
   - `/fix`: 자동 에러 수정
5. **재실행**: 2-4 반복 (보통 3-5회면 완성)

#### 🧪 Copilot 생성 코드 검증 체크리스트

매번 Copilot 제안을 수락하기 전:
- [ ] 이 코드가 정확히 무엇을 하는지 이해했는가?
- [ ] 요구사항을 모두 충족하는가?
- [ ] 에러 처리가 적절한가?
- [ ] 보안 문제는 없는가? (API 키 하드코딩 등)
- [ ] 성능 문제는 없는가?
- [ ] 테스트 가능한 코드인가?

**의심스러우면 Copilot에게 물어보기:**
```
[코드 선택] → Ctrl+I → "/explain 이 코드가 하는 일을 단계별로 설명해줘"
```

### 학습 팁

#### GitHub Copilot 시대의 학습 전략

1. **코드를 타이핑하는 데 시간 쓰지 마라**
   - Copilot이 10분 만에 쓰는 코드를 1시간 타이핑하지 말 것
   - 대신 **설계, 아키텍처, 검증**에 시간 투자

2. **Copilot이 생성한 코드를 반드시 이해하라**
   - 복사-붙여넣기만 하면 시험에서 망한다
   - `/explain` 명령어 적극 활용
   - 핵심 로직은 반드시 이해하고 넘어가기

3. **Copilot Chat 히스토리를 저장하라**
   - VS Code에서 Chat 우클릭 → Export
   - 나중에 복습 자료로 활용
   - 과제 제출 시 협업 증거로 활용

4. **Slack에서 Copilot 활용법 공유하라**
   - "이런 주석이 효과적이었어요" 공유
   - "이 명령어 조합이 좋더라" 공유
   - 다른 학생의 Copilot 활용 패턴 배우기

5. **프로젝트 포트폴리오를 GitHub에 구축하라**
   - Copilot과 협업한 프로젝트도 훌륭한 포트폴리오
   - README에 "GitHub Copilot 활용 방법" 섹션 추가
   - 채용 담당자들도 AI 협업 능력을 중요하게 본다

### 흔한 실수와 해결책

| 실수 | 문제 | 해결책 |
|------|------|--------|
| Copilot 맹신 | 생성 코드를 검증 없이 사용 | 항상 실행하고 `/explain`으로 확인 |
| 모호한 주석 | "함수 만들어줘" | 입출력, 조건, 제약사항 명시 |
| 컨텍스트 부족 | Copilot이 프로젝트 구조 몰라서 엉뚱한 코드 | 관련 파일 열어두기, 주석에 컨텍스트 제공 |
| 일회성 사용 | 한 번 생성하고 끝 | Chat으로 3-5회 개선 요청 |
| 학습 포기 | "Copilot이 다 하니까 나는 안 배워도 되겠네" | `/explain`으로 이해 필수, 설계는 사람 몫 |

### 주차별 자기평가 체크리스트

**1-3주차:**
- [ ] GitHub Student Pack 신청 완료했는가?
- [ ] Copilot을 매일 사용하는가?
- [ ] `/explain`, `/fix`, `/tests` 명령어를 써봤는가?
- [ ] MCP 서버를 Copilot과 만들어봤는가?

**4-7주차:**
- [ ] 복잡한 요구사항을 Copilot에게 전달할 수 있는가?
- [ ] Copilot 생성 코드의 문제점을 스스로 발견하는가?
- [ ] Chat으로 5회 이상 대화하며 코드 개선해봤는가?

**8-13주차:**
- [ ] GraphRAG 시스템을 Copilot과 구축할 수 있는가?
- [ ] 전체 프로젝트 아키텍처를 Copilot에게 요청할 수 있는가?
- [ ] Copilot 없이도 기본 Agent는 만들 수 있는가? (중요!)

**14-16주차:**
- [ ] 캡스톤 프로젝트를 Copilot과 효율적으로 협업하는가?
- [ ] Copilot 활용으로 개발 속도가 3배 이상 빨라졌는가?
- [ ] Copilot 시대의 개발자로서 경쟁력을 갖췄는가?

---

## 📞 강의 운영

### 강의 방식
- **이론 강의**: 개념 설명, 아키텍처 분석, 사례 연구
- **실습 세션**: 강사 시연 후 학생 직접 구현
- **코드 리뷰**: 주요 과제에 대한 피드백 세션
- **세미나**: 최신 논문 및 기술 동향 발표

### 소통 채널
- **Slack 워크스페이스**: 실시간 질의응답 및 공지
- **GitHub Organization**: 코드 공유 및 협업
- **Notion**: 강의 자료 및 참고 문서
- **Office Hour**: 주 1회 대면/온라인 상담

### 팀 구성
- **팀 크기**: 2-3명
- **역할 분담**: PM, Developer, QA (순환 권장)
- **협업 도구**: GitHub Projects, Notion

---

## 🚀 AI 시대의 개발자: GitHub Copilot과 함께

### 이 강좌가 전통적인 코딩 수업과 다른 점

**전통적 방식:**
```
1. 교수가 문법 설명
2. 학생이 처음부터 타이핑
3. 에러 발생 시 혼자 디버깅
4. 주말 내내 코딩
5. 결과물: 200줄 코드
```

**본 강좌 방식 (GitHub Copilot 활용):**
```
1. Copilot Chat에 요구사항 설명
2. Copilot이 1분 만에 초안 생성
3. /explain으로 코드 이해
4. /fix로 에러 해결
5. Chat으로 개선 요청
6. 2시간 만에 완성 → 남은 시간은 설계·테스트·문서화
7. 결과물: 1000줄 코드 + 더 나은 아키텍처
```

### 2026년 현실: GitHub Copilot 없이는 경쟁 불가

**통계로 보는 Copilot의 영향:**
- **GitHub 공식 통계**: Copilot 사용자 55% 생산성 향상
- **개발 속도**: 코드 작성 속도 2-3배 증가
- **Fortune 500 기업**: 92%가 AI 코딩 도구 도입
- **신입 개발자 채용 기준**: "AI 도구 활용 능력" 필수화
- **대학생 특권**: GitHub Student Pack으로 Copilot Pro 무료

**이 강좌를 이수하면:**
- Copilot 없는 개발자보다 **5-10배 빠르게** 코드 작성
- 설계와 검증에 집중하여 **더 나은 품질** 달성
- MCP 표준으로 **어떤 시스템이든 Agent와 통합** 가능
- **무료 도구로 프로 수준 개발** 가능

### GitHub Copilot 활용의 핵심

#### 무엇이 바뀌었나?

**Before Copilot:**
- 구글링 → StackOverflow → 복붙 → 수정 → 에러 → 반복
- 10줄 함수 작성에 10분

**With Copilot:**
- 주석 작성 → Tab → 완성 (10초)
- 에러 발생 → `/fix` → 해결 (30초)
- 리팩토링 필요 → Chat → 개선 (1분)

#### 무엇을 배워야 하나?

**타이핑 속도** ❌ → **프롬프트 작성 능력** ✅
**문법 암기** ❌ → **코드 리뷰 능력** ✅  
**혼자 디버깅** ❌ → **Copilot과 협업** ✅
**코드 많이 쓰기** ❌ → **좋은 설계하기** ✅

### 윤리적 고려사항

#### GitHub Copilot 생성 코드의 책임

- **Copilot이 생성한 코드도 내 책임**: 버그, 보안 취약점, 라이선스 위반
- **저작권**: Copilot 학습 데이터 관련 논란 존재, 하지만 생성 코드 사용은 합법
- **표절**: Copilot 사용은 표절 아님. 단, 코드 이해는 필수
- **학습 목적**: 과제에서 Copilot 사용 시 반드시 협업 로그 제출

#### 본 강좌의 원칙

1. **Copilot 사용을 숨기지 않는다**: 과제 제출 시 Chat 히스토리 필수
2. **Copilot 생성 코드를 이해한다**: `/explain`으로 학습 필수
3. **Copilot에 의존하되 맹신하지 않는다**: 검증은 사람의 책임
4. **Copilot과 경쟁이 아닌 협업**: AI는 도구이자 파트너

### 졸업 후 진로 전망

#### GitHub Copilot 시대에 더 중요해진 역량

1. **시스템 설계 능력**: Copilot이 코드는 쓰지만, 아키텍처는 사람이 설계
2. **요구사항 분석**: 비즈니스 문제를 Copilot이 이해할 수 있는 명세로 변환
3. **코드 리뷰**: Copilot 생성 코드의 품질을 판단하는 안목
4. **통합 능력**: 여러 시스템을 연결하는 능력 (MCP의 중요성)
5. **문제 해결**: Copilot이 모르는 것을 해결하는 창의력

#### 포트폴리오 구축 전략 (GitHub Copilot 활용)

강좌 종료 시 GitHub에 다음을 공개:
- **캡스톤 프로젝트** (MCP 기반, 프로덕션 레벨)
  - README에 "GitHub Copilot으로 개발 속도 5배 향상" 명시
- **3개 이상의 MCP 서버** (Copilot과 개발)
- **프레임워크 활용 사례** (LangGraph/CrewAI/AutoGen)
- **README에 "Copilot 활용 방법" 섹션**:
  ```markdown
  ## GitHub Copilot 활용
  
  이 프로젝트는 GitHub Copilot을 활용하여 개발되었습니다.
  
  ### 주요 활용 사례
  - MCP 서버 구조: Copilot Chat으로 초안 생성
  - 에러 처리: /fix 명령어로 자동 수정
  - 테스트 코드: /tests 명령어로 pytest 생성
  
  ### 개발 효율
  - 예상 개발 시간: 40시간
  - 실제 개발 시간: 12시간 (Copilot 덕분)
  - 생산성 향상: 3.3배
  ```
- **시연 영상** (YouTube/Loom)

### 지속적 학습 로드맵

**강좌 종료 후 1-3개월:**
- GitHub Copilot으로 개인 프로젝트 5개 구축
- MCP 서버를 오픈소스로 공개
- Copilot 활용 팁을 블로그에 작성

**3-6개월:**
- "GitHub Copilot과 함께한 AI Agent 개발" 블로그 시리즈
- Kaggle 대회에 Agent 적용 (Copilot 활용)
- 스타트업 인턴십 (Copilot 활용 능력 강조)

**6-12개월:**
- PyCon에서 "Copilot으로 Agent 개발하기" 발표
- Agent 관련 논문 작성 (Copilot이 실험 코드 생성)
- 자체 Agent 제품 출시 (Copilot으로 개발)

**1년 이후:**
- AI Agent 전문가 + Copilot 파워유저
- "Copilot 시대의 개발자" 교육 제공
- Agent 기반 스타트업 창업 (Copilot으로 MVP 빠르게 구축)

---

## 🚀 과정 이수 후 진로

### 취업 가능 분야
- AI/ML Engineer (Agent 전문)
- LLM Application Developer
- AI Solutions Architect
- Research Scientist (Agentic AI)
- DevOps Engineer (MLOps/LLMOps)

### 추가 학습 경로
- Advanced Topics: Reinforcement Learning for Agents
- Specialization: Domain-specific Agent Development
- Research: Agent Safety & Alignment
- Certification: Cloud Platform AI Certifications

---

## 📅 중요 일정

| 일자 | 내용 |
|------|------|
| 1주차 | 오리엔테이션, 팀 구성 |
| 3주차 | MCP 서버 과제 제출 |
| 7주차 | 중간 프로젝트 제출 |
| 12주차 | 캡스톤 프로젝트 중간발표 |
| 15-16주차 | 캡스톤 프로젝트 최종발표 |
| 16주차 | 최종 보고서 및 코드 제출 |

---

## ⚠️ 학습 주의사항

### API 비용 관리
- 각 팀에게 제한된 API 크레딧 제공
- 효율적인 프롬프트 설계 및 캐싱 활용 필수
- 로컬 LLM(Ollama) 활용 권장 (개발 단계)

### 윤리적 고려사항
- Agent가 생성한 내용의 검증 책임
- 개인정보 처리 시 주의사항
- 편향성(Bias) 및 공정성 고려
- 투명성 및 설명가능성 확보

### 보안
- API 키 관리 (환경 변수, Secrets Manager)
- 프롬프트 인젝션 방지
- 출력 검증 및 샌드박싱

---

## 📌 특별 세션

### Guest Lectures (예정)
- **4주차**: LangChain 팀 개발자 (LangGraph 실전 활용)
- **9주차**: GraphRAG 실무자 (지식 그래프 구축 사례)
- **14주차**: 스타트업 CTO (Agent 기반 제품 개발)

### Hackathon
- **8주차 주말**: 24시간 Agent Hackathon
- 주제: MCP를 활용한 창의적 Agent 개발
- 우수팀 시상 및 발표 기회

---

## 🏆 학습 성과 인증

### 이수 시 제공
- 강좌 이수증
- GitHub 포트폴리오 (프로젝트 코드)
- 프로젝트 시연 영상
- 추천서 (우수 학생 대상)

### 추가 기회
- 우수 프로젝트 학회 발표 지원
- 산학 협력 프로젝트 연계
- 인턴십 추천

---

## 📧 문의

- **담당 교수**: [교수명]
- **이메일**: [이메일]
- **연구실**: [위치]
- **상담 시간**: [요일 시간]

---

**이 강좌는 2026년 1월 기준 최신 AI Agent 기술 트렌드를 반영하여 설계되었으며, 기술 발전에 따라 내용이 업데이트될 수 있습니다.**

*Last Updated: 2026-01-02*
