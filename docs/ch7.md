# 제7장: 멀티에이전트 시스템: 프레임워크 비교와 A2A 실전

## 학습 목표

1. 멀티에이전트 시스템이 필요한 상황을 식별한다.
2. 주요 프레임워크(LangGraph, OpenAI Agents SDK, Google ADK, CrewAI, AutoGen)의 설계 철학과 차이를 비교한다.
3. A2A 프로토콜을 활용하여 이종 프레임워크 에이전트를 연동하는 방법을 설명한다.
4. 로우코드/노코드 도구와 코드 기반 프레임워크의 선택 기준을 판단한다.

## 선수 지식

- 6장에서 구현한 LangGraph 워크플로우 개념
- Python TypedDict와 상태 관리
- OpenAI API 사용 경험

---

## 7.1 멀티에이전트 시스템이란?

6장에서 구현한 LangGraph 워크플로우는 단일 LLM이 여러 단계를 수행했다. 그러나 작업이 복잡해지면 하나의 에이전트가 모든 역할을 수행하는 것은 비효율적일 수 있다. 멀티에이전트 시스템은 여러 AI 에이전트가 각자의 전문 역할을 맡아 협력하는 구조다.

단일 에이전트의 한계는 다음과 같다. 첫째, 컨텍스트 윈도우에 모든 정보를 담기 어렵다. 둘째, 한 번의 프롬프트로 여러 전문 역할을 수행하면 품질이 떨어질 수 있다. 셋째, 복잡한 작업에서 LLM이 단계를 건너뛰거나 환각(hallucination)을 일으킬 가능성이 높아진다.

멀티에이전트는 이러한 한계를 극복한다. 연구자 에이전트는 정보를 수집하고, 작성자 에이전트는 문서를 작성하며, 검토자 에이전트는 품질을 검증한다. 각 에이전트가 특화된 프롬프트와 역할을 가지므로 전문성이 높아지고, 상호 검증을 통해 환각을 줄일 수 있다.

**표 7.1** 단일 에이전트 vs 멀티에이전트

| 특성 | 단일 에이전트 | 멀티에이전트 |
|-----|-------------|-------------|
| 복잡성 | 낮음 | 높음 |
| 전문성 | 일반적 | 역할별 전문화 |
| 컨텍스트 관리 | 단순 | 분산/공유 |
| 비용 | 낮음 | 높음 (API 호출 증가) |
| 적합한 작업 | 단순 Q&A, 도구 호출 | 복잡한 워크플로우, 다단계 분석 |

---

## 7.2 멀티에이전트 아키텍처 패턴

멀티에이전트 시스템은 에이전트 간 상호작용 방식에 따라 여러 패턴으로 분류된다.

### 계층형(Hierarchical)

관리자 에이전트가 작업을 분배하고 하위 에이전트들이 실행한다. 복잡한 작업을 분해하여 전문 에이전트에게 위임하는 방식이다. 관리자가 결과를 수집하고 최종 출력을 구성한다. 이 패턴은 명확한 위계가 필요한 프로젝트 관리, 고객 서비스 에스컬레이션 등에 적합하다.

### 협력형(Collaborative)

에이전트들이 동등한 위치에서 협력한다. 각 에이전트가 자신의 전문 분야를 담당하고, 결과물을 다음 에이전트에게 전달한다. 본 장의 실습에서 구현하는 "연구자 → 작성자 → 검토자" 파이프라인이 이 패턴에 해당한다.

### 경쟁형(Adversarial)

에이전트들이 서로의 결과를 비판하고 검증한다. 한 에이전트가 주장을 하면 다른 에이전트가 반박하는 형태다. 법률 문서 검토, 논쟁적 주제 분석 등에서 다양한 관점을 확보하는 데 유용하다.

### 순차형(Sequential)

에이전트들이 파이프라인 형태로 순서대로 작업을 수행한다. 앞 에이전트의 출력이 다음 에이전트의 입력이 된다. 문서 생성, 번역, 요약 등 단계가 명확한 작업에 적합하다.

---

## 7.3 프레임워크 비교: 2026년 기준

멀티에이전트 시스템을 구현하는 프레임워크는 2025년을 기점으로 급격히 다양화되었다. 각 프레임워크의 설계 철학과 차이를 이해해야 프로젝트에 적합한 선택을 할 수 있다.

### LangGraph

6장에서 학습한 것처럼 노드와 엣지로 에이전트 간 흐름을 명시적으로 정의한다. 조건 분기와 순환 구조를 자유롭게 표현할 수 있어 복잡한 워크플로우에 적합하다. 1.0 릴리스(2025.10)로 API 안정성이 보장되며, Store API를 통한 장기 메모리도 지원한다.

### OpenAI Agents SDK

5장에서 소개한 OpenAI Agents SDK는 핸드오프(handoff) 메커니즘으로 멀티에이전트를 구현한다. 에이전트 간 위임을 `transfer_to_<agent_name>` 도구로 처리하며, `on_handoff` 콜백으로 위임 시점의 컨텍스트를 가공할 수 있다. 코드량이 적고 직관적이지만, LangGraph 수준의 세밀한 상태 제어는 어렵다.

### Google ADK (Agent Development Kit)

Google이 2025년 4월 Cloud Next에서 공개한 ADK는 세 가지 워크플로우 에이전트를 제공한다. `SequentialAgent`는 하위 에이전트를 순차 실행하고, `ParallelAgent`는 동시 실행하며, `LoopAgent`는 반복 실행한다. `sub_agents` 매개변수로 트리 구조의 계층적 멀티에이전트를 구성할 수 있다. Gemini 모델에 최적화되어 있으나 다른 모델도 지원한다.

### CrewAI

역할 기반 에이전트 설계를 강조한다. 에이전트에게 명확한 역할(role), 목표(goal), 배경(backstory)을 부여하고, 태스크 단위로 작업을 정의한다. YAML 기반 설정을 지원하여 코드 없이 에이전트를 정의할 수 있다. 빠른 프로토타이핑에 적합하며 학습 곡선이 낮다.

### AutoGen v0.4 / AG2

Microsoft Research의 AutoGen은 v0.4(2025.1)에서 비동기 이벤트 기반 액터 모델로 완전 재설계되었다. 3계층 API(AgentChat/Core/Extensions)로 구성되며, Magentic-One은 5개 전문 에이전트(Orchestrator, WebSurfer, FileSurfer, Coder, ComputerTerminal)가 Task Ledger/Progress Ledger로 협업하는 범용 멀티에이전트 시스템이다. 한편, AutoGen 0.2 아키텍처를 유지하는 AG2 포크가 별도로 진행 중이다.

**표 7.2** 멀티에이전트 프레임워크 비교 (2026년 기준)

| 기준 | LangGraph | Agents SDK | Google ADK | CrewAI | AutoGen v0.4 |
|-----|-----------|------------|------------|--------|-------------|
| 설계 철학 | 그래프 워크플로우 | 핸드오프 기반 | 계층적 에이전트 트리 | 역할 기반 팀 | 이벤트 기반 액터 |
| 멀티에이전트 방식 | 노드·엣지 | transfer_to | sub_agents | Crew/Task | GroupChat/Teams |
| 학습 곡선 | 높음 | 낮음 | 중간 | 낮음 | 높음 |
| 상태 제어 | 매우 세밀 | 제한적 | 중간 | 추상화 | 중간 |
| 모델 종속성 | 없음 | OpenAI 우선 | Gemini 우선 | 없음 | 없음 |
| 적합한 사용 | 복잡한 워크플로우 | 빠른 위임 체인 | Google 생태계 | 빠른 프로토타입 | 대화형 협업 |

---

## 7.4 Human-in-the-loop 설계

고위험 작업에서는 에이전트의 결정을 사람이 검토하고 승인해야 한다. Human-in-the-loop는 자동화와 인간 판단을 결합하는 설계 패턴이다.

### 승인이 필요한 작업

모든 작업에 인간 승인이 필요한 것은 아니다. 다음과 같은 작업에서 승인 단계를 추가한다.

- 외부 시스템에 영향을 미치는 작업 (이메일 전송, 결제 처리)
- 되돌리기 어려운 작업 (데이터 삭제, 계정 변경)
- 민감한 정보를 다루는 작업 (개인정보, 비밀)
- 비용이 높은 작업 (대량 API 호출, 서버 프로비저닝)

### 승인 요청 UX 설계

승인 요청은 명확하고 간결해야 한다. 에이전트가 수행하려는 작업, 예상 결과, 위험 요소를 제시하고, 승인/거부/수정 옵션을 제공한다. 타임아웃을 설정하여 무한 대기를 방지하고, 타임아웃 시 안전한 기본값(예: 거부)으로 폴백한다.

### 프레임워크 지원

AutoGen은 `human_input_mode` 파라미터로 인간 개입 시점을 제어한다. `ALWAYS`로 설정하면 매 단계마다 승인을 요청하고, `TERMINATE`는 종료 전에만 확인한다. LangGraph에서는 `interrupt_before` 파라미터로 특정 노드 실행 전 중단하고 인간 입력을 받을 수 있다.

---

## 7.5 A2A 실전: 이종 프레임워크 에이전트 연동

4장에서 A2A 프로토콜의 개념을 소개했다. 이 절에서는 실제로 서로 다른 프레임워크로 만든 에이전트를 A2A로 연동하는 방법을 다룬다.

### 7.5.1 에이전트 카드(Agent Card)

A2A에서 에이전트의 신원과 능력을 선언하는 JSON 문서가 에이전트 카드다. `/.well-known/agent.json` 경로에 게시하면, 클라이언트 에이전트가 이를 발견하고 어떤 작업을 위임할 수 있는지 판단한다.

```json
{
  "name": "weather-agent",
  "version": "1.0",
  "url": "https://weather.example.com",
  "capabilities": {"streaming": true},
  "skills": [{"id": "forecast", "name": "날씨 예보"}]
}
```

에이전트 카드에는 이름, 버전, URL, 지원 능력(capabilities), 기술 목록(skills)을 명시한다. 기술(skill)은 에이전트가 수행할 수 있는 구체적인 작업 단위를 의미한다.

### 7.5.2 태스크 위임 흐름

클라이언트 에이전트가 리모트 에이전트에 작업을 위임하는 흐름은 다음과 같다. 클라이언트가 `SendMessage`를 호출하면 리모트가 Task 객체를 반환한다. 클라이언트는 태스크 상태(submitted → working → completed/failed/input_required)를 폴링, 스트리밍, 또는 푸시 방식으로 추적한다. 태스크가 `input_required` 상태가 되면 추가 정보를 보내 대화를 계속할 수 있다.

### 7.5.3 이종 프레임워크 연동

A2A의 핵심 가치는 프레임워크에 구속되지 않는 에이전트 간 통신이다. 예를 들어, Google ADK로 만든 오케스트레이터가 LangGraph 에이전트에게 분석 작업을, CrewAI 에이전트에게 보고서 작성을 위임할 수 있다. 각 에이전트는 내부 구현과 무관하게 A2A의 표준 인터페이스(에이전트 카드 + 태스크 프로토콜)만 준수하면 된다.

이는 MCP가 "에이전트와 도구"의 표준이라면, A2A는 "에이전트와 에이전트"의 표준이라는 점에서 상호 보완적이다. 실무에서는 ADK(개발) + MCP(도구 연동) + A2A(에이전트 간 통신)의 3층 구조가 형성된다.

---

## 7.6 로우코드/노코드 대안: 언제 시각적 도구가 적합한가

코드 기반 프레임워크가 유일한 선택지는 아니다. 로우코드/노코드 도구는 시각적 인터페이스로 에이전트 워크플로우를 구성하며, 특정 상황에서 코드 기반보다 효율적이다.

### 7.6.1 주요 도구

**n8n**은 GitHub 스타 17만 이상의 범용 워크플로우 자동화 플랫폼이다. 400개 이상의 통합 노드를 제공하며, MCP 지원, 네이티브 Python 실행, Redis/Vector 메모리 노드를 갖추고 있다. 기존 SaaS 서비스를 연결하는 "신경계" 역할에 강하다.

**Dify**는 GitHub 스타 12만 이상의 AI 네이티브 앱 개발 플랫폼이다. LLM 오케스트레이션, 비주얼 프롬프트 스튜디오, RAG 내장, 원클릭 배포를 지원한다. LLM 추론을 중심으로 하는 "두뇌" 역할에 특화되어 있다.

두 도구는 경쟁 관계가 아니라 보완 관계다. n8n으로 외부 시스템을 연결하고, Dify로 AI 추론 파이프라인을 구축한 뒤, API/웹훅으로 통합하는 패턴이 실무에서 사용된다.

### 7.6.2 선택 기준

**표 7.3** 코드 기반 vs 로우코드/노코드

| 기준 | 코드 기반 프레임워크 | 로우코드/노코드 |
|------|-------------------|---------------|
| 적합한 팀 | 개발 역량 있는 팀 | 다양한 기술 수준의 팀 |
| 프로토타이핑 속도 | 중간~느림 | 매우 빠름 |
| 제어 수준 | 세밀 | 제한적 |
| 프로덕션 적합성 | 높음 | 중간 |
| 디버깅 | 코드 수준 추적 | 시각적 플로우 추적 |
| 규정 준수/감사 | 코드 리뷰 가능 | 플랫폼 의존적 |

빠른 프로토타이핑, 통합 중심 워크플로우, 비개발자 참여가 중요한 경우 로우코드가 적합하다. 프로덕션 안정성, 세밀한 상태 제어, 규정 준수가 우선인 경우 코드 기반 프레임워크를 선택한다.

---

## 7.7 실습: 단일 에이전트 vs 멀티에이전트 비교

### 실습 목표

1. 동일한 작업(기술 문서 작성)을 단일 에이전트로 구현한다.
2. 같은 작업을 멀티에이전트(연구자, 작성자, 검토자)로 구현한다.
3. 두 접근법의 결과물, 실행 시간, API 호출 수를 비교한다.

### 실습 환경 설정

```bash
cd practice/chapter7
python3 -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r code/requirements.txt
cp code/.env.example code/.env  # OPENAI_API_KEY 설정
```

### 단일 에이전트 구현

단일 에이전트는 하나의 프롬프트로 조사, 작성, 검토를 모두 수행한다.

```python
prompt = f"""당신은 기술 문서 작성 전문가입니다.
주제: {topic}
문서 요구사항: 개념 설명, 코드 예시, 모범 사례
"""
response = llm.invoke(prompt)
```

_전체 코드는 practice/chapter7/code/7-5-single-agent.py 참고_

### 멀티에이전트 구현

멀티에이전트는 세 개의 전문 에이전트가 순차적으로 협력한다. LangGraph의 StateGraph를 사용하여 워크플로우를 정의한다.

```python
workflow.add_node("researcher", researcher_node)
workflow.add_node("writer", writer_node)
workflow.add_node("reviewer", reviewer_node)
```

_전체 코드는 practice/chapter7/code/7-5-multi-agent.py 참고_

### 실행 및 비교

두 스크립트를 각각 실행하고 결과를 비교한다.

```bash
python3 code/7-5-single-agent.py
python3 code/7-5-multi-agent.py
python3 code/7-5-compare.py
```

### 실행 결과

**표 7.4** 단일 에이전트 vs 멀티에이전트 비교 결과

| 항목 | 단일 에이전트 | 멀티에이전트 |
|-----|-------------|-------------|
| API 호출 | 1회 | 3회 |
| 소요 시간 | 14.76초 | 33.31초 |
| 문서 길이 | 2,083자 | 2,146자 |
| 수정 횟수 | - | 1회 |

멀티에이전트는 API를 3배 호출하고 시간이 2.26배 소요되었다. 그러나 연구, 작성, 검토 단계가 분리되어 있어 각 역할에 특화된 프롬프트를 사용할 수 있다. 검토자가 피드백을 제공하면 작성자가 수정하는 반복 구조도 가능하다.

### 실습 산출물

- `practice/chapter7/data/output/ch07_single_result.json`: 단일 에이전트 결과
- `practice/chapter7/data/output/ch07_multi_result.json`: 멀티에이전트 결과
- `practice/chapter7/data/output/ch07_comparison.json`: 비교 분석
- `practice/chapter7/data/output/ch07_document_single.txt`: 단일 에이전트 생성 문서
- `practice/chapter7/data/output/ch07_document_multi.txt`: 멀티에이전트 생성 문서

---

## 7.8 선택 가이드: 언제 무엇을 사용할 것인가

### 단일 에이전트를 선택해야 하는 경우

- 작업이 단순하고 범위가 명확하다
- 빠른 응답이 필요하다
- 비용을 최소화해야 한다
- 문서 요약, 이메일 응답, 특정 정보 검색 등

### 멀티에이전트가 효과적인 경우

- 복잡한 다단계 작업이다
- 환각 감소를 위해 상호 검증이 필요하다
- 긴 컨텍스트를 분산 처리해야 한다
- 병렬 처리로 속도를 향상해야 한다

### 프레임워크 선택 체크리스트

1. **빠른 프로토타입이 필요한가?** → CrewAI
2. **대화형 협업이 핵심인가?** → AutoGen
3. **복잡한 조건 분기가 필요한가?** → LangGraph
4. **Human-in-the-loop가 중요한가?** → AutoGen 또는 LangGraph
5. **기존 LangChain 코드가 있는가?** → LangGraph

---

## 7.9 실패 사례와 교훈

### 에이전트 간 무한 루프

두 에이전트가 서로에게 작업을 위임하며 무한히 반복하는 상황이 발생할 수 있다. 이를 방지하려면 최대 반복 횟수를 설정하고, 각 에이전트의 역할과 종료 조건을 명확히 정의해야 한다.

### 과도한 API 비용

멀티에이전트는 단일 에이전트 대비 API 호출이 증가한다. 실습 결과에서도 3배의 API 호출이 발생했다. 프로덕션 환경에서는 캐싱, 더 저렴한 모델 사용, 필수적인 경우에만 에이전트 투입 등의 전략이 필요하다.

### 불명확한 역할 분담

에이전트 간 역할이 중복되면 같은 작업을 반복하거나 서로 충돌하는 결과를 생성한다. 각 에이전트의 책임을 명확히 정의하고, 출력 형식을 표준화해야 한다.

---

## 핵심 정리

- 멀티에이전트 시스템은 전문 역할을 분담하여 복잡한 작업을 수행한다.
- 계층형, 협력형, 경쟁형, 순차형 등 다양한 아키텍처 패턴이 있다.
- LangGraph(그래프), Agents SDK(핸드오프), Google ADK(계층 트리), CrewAI(역할), AutoGen(이벤트)은 각각 다른 설계 철학을 가진다.
- A2A 프로토콜로 이종 프레임워크 에이전트를 연동할 수 있다(에이전트 카드 + 태스크 위임).
- n8n, Dify 등 로우코드 도구는 빠른 프로토타이핑과 통합 중심 워크플로우에 적합하다.
- Human-in-the-loop로 고위험 작업에서 인간 승인을 받는다.

---

## 다음 장 예고

8장에서는 RAG의 기본과 에이전트 메모리 아키텍처를 다룬다. 검색 증강 생성(RAG)으로 외부 지식을 활용하는 방법과 함께, 에이전트 메모리 분류 체계(Working, Episodic, Semantic, Procedural)와 RAG-메모리 상호보완 관계를 살펴본다.

---

## 참고문헌

LangChain. (2025). *LangGraph Documentation*. https://docs.langchain.com/oss/python/langgraph

OpenAI. (2025). *OpenAI Agents SDK - Handoffs*. https://openai.github.io/openai-agents-python/handoffs/

Google. (2025). *Agent Development Kit - Multi-Agent Systems*. https://google.github.io/adk-docs/agents/multi-agents/

CrewAI. (2025). *CrewAI Documentation*. https://docs.crewai.com/

Microsoft. (2025). *AutoGen Reimagined: Launching AutoGen 0.4*. https://devblogs.microsoft.com/autogen/autogen-reimagined-launching-autogen-0-4/

A2A Protocol. (2025). *Agent-to-Agent Protocol Specification*. https://a2a-protocol.org/latest/specification/

Google. (2025). *Intro to A2A: Purchasing Concierge Codelab*. https://codelabs.developers.google.com/intro-a2a-purchasing-concierge

n8n. (2025). *n8n - Workflow Automation*. https://github.com/n8n-io/n8n

Dify. (2025). *Dify - AI Application Development Platform*. https://github.com/langgenius/dify

ZenML. (2025). *Google ADK vs LangGraph*. https://www.zenml.io/blog/google-adk-vs-langgraph
