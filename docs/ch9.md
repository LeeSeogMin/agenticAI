# 제9장: GraphRAG/LightRAG/KAG — 관계 기반 추론으로 확장

## 학습 목표

1. 벡터 RAG의 한계와 그래프 기반 접근의 필요성을 이해한다.
2. 지식 그래프의 기본 개념(엔티티, 관계, 트리플)을 학습한다.
3. GraphRAG와 LightRAG의 아키텍처와 차이점을 비교한다.
4. 동일 질의로 벡터 RAG와 그래프 RAG의 결과를 비교 분석한다.
5. 그래프 기반 RAG의 적합한 사용 사례를 식별한다.

## 선수 지식

- 8장에서 구현한 벡터 RAG 개념
- 임베딩과 벡터 유사도 검색
- LangChain 기본 사용법

---

## 9.1 벡터 RAG의 한계

8장에서 구현한 벡터 RAG는 많은 상황에서 효과적이지만, 특정 유형의 질문에서는 한계를 보인다.

### 단순 유사도 매칭의 한계

벡터 검색은 쿼리와 문서 청크 간의 의미적 유사도를 계산한다. "asyncio.gather()의 사용법"처럼 특정 개념을 직접 묻는 질문에는 잘 작동한다. 그러나 "데이터에서 가장 중요한 5가지 테마는 무엇인가?"와 같은 전역적 질문에는 적합하지 않다. 쿼리를 올바른 정보로 안내할 방향성이 없기 때문이다.

### 다중 홉 질문의 어려움

"A가 B에 영향을 주고, B가 C에 영향을 준다면, A와 C의 관계는?" 같은 다중 홉(multi-hop) 질문은 벡터 검색으로 답하기 어렵다. 각 청크가 독립적으로 검색되므로 여러 청크에 걸친 추론이 필요한 질문에서 성능이 떨어진다.

### 관계 정보의 부재

벡터 임베딩은 텍스트의 의미를 포착하지만, 엔티티 간의 명시적 관계는 표현하지 않는다. "코루틴과 태스크의 관계"를 묻는 질문에서, 벡터 RAG는 두 개념이 언급된 청크를 찾을 수는 있지만 그 관계를 직접적으로 추론하지는 못한다.

---

## 9.2 지식 그래프 기초

지식 그래프는 엔티티와 그들 간의 관계를 그래프 구조로 표현한다. 벡터 RAG의 한계를 극복하는 핵심 도구다.

### 엔티티와 관계

엔티티(Entity)는 문서에서 추출된 핵심 개념, 함수, 모듈 등이다. 관계(Relation)는 두 엔티티 간의 연결을 설명한다. 예를 들어, "asyncio.create_task()"와 "코루틴" 사이에 "감싼다"라는 관계가 있다.

### 트리플 구조

지식 그래프의 기본 단위는 트리플(Triple)이다. 주어-술어-목적어 형태로 표현된다.

```
(asyncio.create_task) --[감싼다]--> (코루틴)
(이벤트 루프) --[관리한다]--> (비동기 작업)
```

### LLM 기반 추출

과거에는 규칙 기반이나 통계적 방법으로 엔티티와 관계를 추출했다. 현재는 LLM을 활용하여 더 정확하고 맥락을 이해하는 추출이 가능하다. 프롬프트로 추출 형식을 지정하고, LLM이 JSON 형태로 결과를 반환한다.

```python
result = extract_entities_and_relations(chunk, llm)
# {"entities": [...], "relations": [...]}
```

_전체 코드는 practice/chapter9/code/9-6-graph-rag.py 참고_

---

## 9.3 GraphRAG: Microsoft의 접근

Microsoft가 개발한 GraphRAG는 대규모 텍스트 데이터셋에서 지식 그래프를 자동으로 구축하고 활용하는 시스템이다. 2024년 GitHub에 공개되어 2025년까지 활발히 발전하고 있다.

### 아키텍처 개요

GraphRAG는 두 단계로 작동한다. 먼저 인덱싱 단계에서 소스 문서로부터 엔티티 지식 그래프를 추출한다. 그 다음 관련 엔티티 그룹(커뮤니티)을 탐지하고, 각 커뮤니티에 대한 요약을 사전 생성한다.

쿼리 시에는 각 커뮤니티 요약을 사용하여 부분 응답을 생성한 후, 이를 통합하여 최종 응답을 만든다.

### 글로벌 검색 vs 로컬 검색

GraphRAG는 두 가지 검색 전략을 제공한다. 글로벌 검색은 "데이터셋의 주요 테마는?"처럼 전체 데이터를 아우르는 질문에 적합하다. 로컬 검색은 특정 엔티티나 관계에 대한 상세 질문에 사용된다.

### 트레이드오프

GraphRAG의 주요 단점은 인덱싱 비용이다. 모든 문서에서 엔티티와 관계를 추출하고, 커뮤니티를 탐지하고, 요약을 생성하는 과정에 상당한 LLM 호출이 필요하다. 100만 토큰 규모의 데이터셋에서는 포괄성과 다양성이 크게 향상되지만, 비용과 시간 투자를 정당화할 수 있는지 검토해야 한다.

---

## 9.4 LightRAG: 경량화된 그래프 RAG

LightRAG는 홍콩대학교 데이터과학 연구팀이 개발한 경량 그래프 RAG 프레임워크다. EMNLP2025에서 발표되었으며, GraphRAG의 복잡성을 줄이면서도 성능을 유지한다.

### 핵심 특징

LightRAG는 듀얼 레벨 검색 시스템을 사용한다. 로컬 레벨에서는 세부 정보를 검색하고, 글로벌 레벨에서는 거시적 개념을 검색한다. 두 레벨을 통합하여 포괄적인 검색을 수행한다.

모듈형 설계로 문서 파싱, 인덱스 구축, 검색, 생성 단계가 분리되어 있어 각 모듈을 독립적으로 교체하거나 최적화할 수 있다.

### GraphRAG와의 비교

**표 9.1** GraphRAG vs LightRAG 비교

| 기준 | GraphRAG | LightRAG |
|-----|----------|----------|
| 개발사 | Microsoft | HKUDS |
| 인덱싱 비용 | 높음 | 낮음 |
| 증분 업데이트 | 제한적 | 지원 |
| 검색 전략 | 글로벌/로컬 분리 | 듀얼 레벨 통합 |
| 쿼리 지연 | 표준 | ~30% 감소 |
| 적합한 사용 | 대규모 분석 | 빠른 프로토타입 |

### 성능

LightRAG는 쿼리 지연을 약 30% 감소시킨다(~80ms vs 표준 RAG ~120ms). 벤치마크에서 Naive RAG와 GraphRAG 대비 우수한 성능을 보인다.

---

## 9.5 KAG: 지식 증강 생성

KAG(Knowledge Augmented Generation)는 OpenSPG 엔진 기반의 논리적 추론 프레임워크다. 전문 도메인 지식베이스를 위한 Q&A 솔루션을 제공한다.

### 핵심 특징

KAG는 도메인 특화 지식 그래프를 구축한다. 일반적인 지식 그래프가 아닌 의료, 법률, 금융 등 특정 분야에 최적화된 스키마를 사용한다.

다중 홉 추론을 지원하여 여러 엔티티에 걸친 정보를 연결하고 합성한다. DIKW(Data-Information-Knowledge-Wisdom) 모델을 기반으로 정보를 계층화한다.

### GraphRAG/LightRAG와의 관계

KAG는 벡터 RAG의 모호성과 GraphRAG의 노이즈 문제를 모두 해결하고자 한다. 2025년 업데이트에서 "Lightweight Build" 모드를 도입하여 토큰 비용을 89% 절감했다.

세 프레임워크는 경쟁 관계가 아니라 보완적이다. 동적 환경에서는 RAG, 구조화된 도메인에서는 KAG, 복잡한 관계 추론에서는 GraphRAG/LightRAG가 각각 강점을 보인다.

---

## 9.6 실습: 벡터 RAG vs 그래프 RAG 비교

### 실습 목표

1. 동일한 문서에서 지식 그래프를 추출한다.
2. 벡터 RAG와 그래프 기반 RAG로 동일한 질문에 답변한다.
3. 두 방식의 결과를 비교 분석한다.

### 실습 환경 설정

```bash
cd practice/chapter9
python3 -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r code/requirements.txt
cp code/.env.example code/.env  # OPENAI_API_KEY 설정
```

### 그래프 RAG 실행

```bash
python3 code/9-6-graph-rag.py
python3 code/9-6-compare.py
```

### 실행 결과

**표 9.2** 벡터 RAG vs 그래프 RAG 비교

| 항목 | 벡터 RAG | 그래프 RAG |
|-----|---------|-----------|
| 총 실행 시간 | 21.72초 | 59.00초 |
| 청크/엔티티 수 | 11개 청크 | 41개 엔티티 |
| 관계 수 | - | 44개 |

그래프 RAG는 벡터 RAG 대비 약 2.7배의 시간이 소요되었다. 이는 지식 그래프 구축 과정에서 추가 LLM 호출이 필요하기 때문이다. 그러나 "A와 B의 관계" 같은 관계 중심 질문에서는 그래프 RAG가 더 명확한 답변을 제공한다.

### 질문별 비교 예시

질문: "코루틴과 태스크는 어떤 관계가 있나요?"

그래프 RAG는 "코루틴", "asyncio.create_task", "비동기 작업" 등의 관련 엔티티를 명시적으로 식별하고, 이들 간의 관계를 바탕으로 답변을 구성했다.

### 실습 산출물

- `practice/chapter9/data/output/ch09_knowledge_graph.json`: 추출된 지식 그래프
- `practice/chapter9/data/output/ch09_vector_result.json`: 벡터 RAG 결과
- `practice/chapter9/data/output/ch09_graph_result.json`: 그래프 RAG 결과
- `practice/chapter9/data/output/ch09_comparison.json`: 비교 분석

---

## 9.7 그래프 RAG 선택 가이드

### 벡터 RAG가 적합한 경우

- 특정 정보를 검색하는 단순 질문
- 실시간 응답이 필요한 상황
- 인덱싱 비용을 최소화해야 할 때
- 문서가 자주 업데이트되는 환경

### 그래프 RAG가 필요한 경우

- "전체 데이터의 주요 테마"처럼 전역적 질문
- 엔티티 간 관계를 파악해야 하는 질문
- 다중 홉 추론이 필요한 복잡한 질문
- 도메인 지식의 구조화가 중요한 분야

### 하이브리드 접근

실무에서는 벡터 검색과 그래프 검색을 결합하는 것이 효과적이다. 벡터 검색으로 초기 후보를 찾고, 그래프 순회로 관련 엔티티를 확장한다. LightRAG의 듀얼 레벨 검색이 이러한 하이브리드 접근의 예다.

---

## 9.8 실패 사례와 교훈

### 엔티티/관계 추출 오류

LLM 기반 추출은 완벽하지 않다. 모호한 표현, 암묵적 관계, 도메인 특화 용어에서 오류가 발생한다. 추출 결과를 검증하는 단계를 추가하거나, 도메인 특화 프롬프트를 설계해야 한다.

### 과도한 인덱싱 비용

소규모 데이터셋에 GraphRAG를 적용하면 비용 대비 효과가 낮다. 데이터 규모와 질문 유형을 분석하여 그래프 RAG가 정말 필요한지 평가해야 한다.

### 그래프 품질 저하

오래된 정보로 구축된 그래프는 잘못된 답변을 유도한다. 문서 업데이트 시 그래프도 함께 갱신하는 파이프라인이 필요하다. LightRAG는 증분 업데이트를 지원하여 이 문제를 완화한다.

---

## 핵심 정리

- 벡터 RAG는 전역적 질문, 다중 홉 추론, 관계 파악에 한계가 있다.
- 지식 그래프는 엔티티와 관계를 트리플 형태로 표현한다.
- GraphRAG는 커뮤니티 요약과 계층적 검색으로 전역적 질문에 강하다.
- LightRAG는 듀얼 레벨 검색으로 GraphRAG를 경량화했다.
- 벡터 RAG는 단순 검색, 그래프 RAG는 관계 추론에 각각 강점이 있다.

---

## 다음 장 예고

10장에서는 신뢰성과 환각 방지를 다룬다. 에이전트가 생성하는 정보의 정확성을 검증하고, 잘못된 출력을 방지하는 파이프라인을 구축한다.

---

## 참고문헌

Microsoft. (2025). *GraphRAG: A modular graph-based RAG system*. https://github.com/microsoft/graphrag

Microsoft Research. (2024). *GraphRAG: Unlocking LLM discovery on narrative private data*. https://www.microsoft.com/en-us/research/blog/graphrag-unlocking-llm-discovery-on-narrative-private-data/

HKUDS. (2025). *LightRAG: Simple and Fast Retrieval-Augmented Generation*. https://github.com/HKUDS/LightRAG

OpenSPG. (2025). *KAG: Knowledge Augmented Generation*. https://github.com/OpenSPG/KAG

Edge et al. (2024). *From Local to Global: A Graph RAG Approach to Query-Focused Summarization*. arXiv:2404.16130
