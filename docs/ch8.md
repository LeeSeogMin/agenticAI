# 제8장: RAG의 기본과 에이전트 메모리 아키텍처

## 학습 목표

1. RAG가 필요한 상황과 아키텍처를 이해한다.
2. 에이전트 메모리를 4가지 유형(Working, Episodic, Semantic, Procedural)으로 분류하고 각각의 구현 전략을 설명한다.
3. RAG와 에이전트 메모리의 차이와 상호보완 관계를 판단한다.
4. 답변에 인용/출처를 포함하는 방법을 구현한다.

## 선수 지식

- 7장에서 학습한 멀티에이전트 개념
- Python 기본 문법
- OpenAI API 사용 경험

---

## 8.1 RAG가 필요한 이유

7장에서 학습한 에이전트는 LLM의 내재된 지식만으로 답변을 생성했다. 그러나 LLM에는 명확한 한계가 있다. 학습 데이터의 지식 컷오프 이후 정보를 알지 못하고, 조직 내부 문서나 도메인 특화 지식에 접근할 수 없으며, 출처 없이 답변하므로 검증이 어렵다.

검색 증강 생성(Retrieval Augmented Generation, RAG)은 이러한 한계를 극복한다. RAG는 외부 지식을 검색하여 LLM의 컨텍스트에 주입하고, 이를 바탕으로 답변을 생성하는 하이브리드 아키텍처다. 2025년 기준 프로덕션 AI 애플리케이션의 약 60%가 RAG를 사용한다.

RAG의 핵심 이점은 다음과 같다. 첫째, 문서를 추가하거나 업데이트하면 즉시 지식이 반영된다. 파인튜닝처럼 모델을 재학습할 필요가 없다. 둘째, 답변에 출처를 명시할 수 있어 검증 가능성이 높아진다. 셋째, 외부 지식에 근거하므로 환각(hallucination)이 감소한다.

**표 8.1** RAG vs 파인튜닝 비교

| 기준 | RAG | 파인튜닝 |
|-----|-----|---------|
| 지식 업데이트 | 문서 추가로 즉시 | 재학습 필요 |
| 비용 | 임베딩/검색 비용 | 학습 비용 높음 |
| 출처 추적 | 가능 | 불가능 |
| 환각 감소 | 근거 기반 답변 | 제한적 |
| 적합한 상황 | 자주 변경되는 지식 | 스타일/형식 변경 |

---

## 8.2 RAG 아키텍처 개요

RAG 시스템은 크게 두 단계로 구성된다: 인덱싱(오프라인)과 검색-생성(온라인).

### 인덱싱 단계

문서를 수집하고 처리하여 검색 가능한 형태로 저장한다. 문서를 청크(chunk)로 분할하고, 각 청크를 임베딩 모델로 벡터로 변환한 후, 벡터 데이터베이스에 저장한다.

### 검색-생성 단계

사용자 질문이 들어오면 질문을 벡터로 변환하고, 벡터 데이터베이스에서 유사한 청크를 검색한다. 검색된 청크를 LLM의 컨텍스트에 포함시키고, LLM이 컨텍스트를 참조하여 답변을 생성한다.

### 임베딩과 벡터 유사도

임베딩은 텍스트를 고차원 벡터로 변환하는 과정이다. 의미적으로 유사한 텍스트는 벡터 공간에서 가까운 위치에 놓인다. 코사인 유사도나 유클리드 거리를 사용하여 벡터 간 유사도를 계산한다.

---

## 8.3 벡터 데이터베이스: ChromaDB

벡터 데이터베이스는 고차원 벡터를 효율적으로 저장하고 검색하는 특수 데이터베이스다. ChromaDB는 오픈소스 벡터 데이터베이스로, 로컬 개발과 프로토타이핑에 적합하다.

```python
from langchain_community.vectorstores import Chroma

vector_store = Chroma.from_documents(
    documents=chunks,
    embedding=embeddings
)
```

_전체 코드는 practice/chapter8/code/8-6-rag-basic.py 참고_

### 주요 벡터 데이터베이스 비교

| 데이터베이스 | 특징 | 적합한 사용 |
|------------|------|-----------|
| ChromaDB | 오픈소스, 로컬 실행 | 프로토타이핑, 소규모 |
| Pinecone | 관리형 서비스, 확장성 | 프로덕션, 대규모 |
| Milvus | 오픈소스, 분산 처리 | 엔터프라이즈 |
| Weaviate | 그래프 기능 포함 | 관계 기반 검색 |

---

## 8.4 청킹 전략: 문서를 효과적으로 분할하기

청킹은 RAG 성능에 큰 영향을 미친다. 청크가 너무 작으면 컨텍스트가 부족하고, 너무 크면 관련 없는 정보가 포함되어 검색 정확도가 떨어진다.

### 고정 크기 청킹

RecursiveCharacterTextSplitter는 지정된 크기로 문서를 분할하되, 문장이나 문단 경계를 존중한다. 업계 권장 설정은 청크 크기 256-512 토큰, 오버랩 10-20%다.

```python
splitter = RecursiveCharacterTextSplitter(
    chunk_size=500,
    chunk_overlap=50
)
chunks = splitter.split_documents(documents)
```

### 의미 기반 청킹

의미 기반 청킹은 문장의 임베딩 유사도를 기준으로 분할 지점을 결정한다. 의미적으로 연관된 문장을 같은 청크에 유지하므로 검색 품질이 향상된다. 다만 모든 문장의 임베딩을 계산해야 하므로 비용이 높다.

### 트레이드오프

작은 청크는 쿼리와 정확히 매칭되지만 주변 컨텍스트를 잃는다. 큰 청크는 컨텍스트를 보존하지만 임베딩이 희석된다. 실험을 통해 데이터와 사용 사례에 맞는 최적값을 찾아야 한다.

---

## 8.5 검색 품질 향상

기본 벡터 검색만으로는 충분하지 않을 수 있다. 검색 품질을 향상시키는 기법을 적용한다.

### 하이브리드 검색

키워드 검색(BM25)과 벡터 검색을 결합한다. 벡터 검색은 의미적 유사성을 포착하고, 키워드 검색은 정확한 용어 매칭에 강하다. 두 결과를 융합하여 더 나은 검색 결과를 얻는다.

### 리랭킹

초기 검색 결과를 더 정교한 모델로 재정렬한다. Cross-encoder 모델이 쿼리와 각 문서의 관련성을 직접 평가하여 순위를 조정한다. 계산 비용이 높으므로 Top-K 결과에만 적용한다.

### 쿼리 확장

사용자 쿼리를 여러 형태로 확장하여 검색한다. LLM을 사용하여 쿼리를 재작성하거나, 동의어를 추가하여 검색 범위를 넓힌다.

---

## 8.6 인용과 출처 추적

RAG의 핵심 장점 중 하나는 출처를 명시할 수 있다는 점이다. 답변에 출처를 포함하면 신뢰성이 높아지고 검증이 가능해진다.

### 청크 메타데이터 관리

각 청크에 원본 문서 정보, 청크 ID, 페이지 번호 등의 메타데이터를 저장한다. 검색 결과와 함께 메타데이터를 반환하여 출처를 추적한다.

```python
chunk.metadata["source_name"] = Path(source_path).name
chunk.metadata["chunk_id"] = i
```

### 답변에 출처 포함

프롬프트에서 LLM에게 출처를 명시하도록 지시한다. 검색된 문서에 출처 레이블을 붙이고, LLM이 이를 참조하여 답변에 인용을 포함한다.

---

## 8.7 에이전트 메모리 분류 체계

RAG가 "외부 지식을 검색하여 답변하는 구조"라면, 에이전트 메모리는 "에이전트가 경험을 축적하고 활용하는 구조"다. 인지과학의 메모리 분류를 빌려, 에이전트 메모리를 네 가지 유형으로 나눌 수 있다.

### 8.7.1 Working Memory (작업 메모리)

현재 컨텍스트 윈도우에 담긴 정보다. LLM에 전달되는 시스템 프롬프트, 대화 히스토리, 검색 결과가 여기에 해당한다. 가장 직접적이지만 토큰 한도에 의해 용량이 제한된다. 6장의 LangGraph State 객체도 하나의 워크플로우 내 작업 메모리로 볼 수 있다.

### 8.7.2 Episodic Memory (일화 메모리)

과거 이벤트와 결과를 시간순으로 저장한다. "어제 사용자가 파이썬 비동기 프로그래밍에 대해 질문했고, asyncio.gather()를 추천했다"와 같은 구체적 경험이 해당한다. 에이전트가 이전 상호작용의 맥락을 기억하여 일관된 응답을 제공하는 데 활용된다.

### 8.7.3 Semantic Memory (의미 메모리)

일화에서 일반화된 지식이다. "이 사용자는 한국어를 선호하고, 코드 예제를 중시한다"와 같은 패턴화된 정보가 해당한다. 특정 이벤트에 종속되지 않으므로 다양한 상황에 적용할 수 있다.

### 8.7.4 Procedural Memory (절차 메모리)

학습된 도구 사용 패턴과 작업 수행 방법이다. "날씨 질문에는 weather_tool을 호출하고, 결과를 한국어로 요약한다"와 같은 행동 규칙이 해당한다. 프롬프트 패턴이나 파인튜닝으로 모델에 내재화할 수 있다.

**표 8.2** 에이전트 메모리 4가지 유형

| 유형 | 내용 | 지속성 | 예시 |
|------|------|--------|------|
| Working | 현재 컨텍스트 | 세션 내 | 대화 히스토리, 검색 결과 |
| Episodic | 과거 이벤트 | 장기 | "어제 질문에 X를 추천했다" |
| Semantic | 일반화된 지식 | 장기 | "사용자는 한국어를 선호한다" |
| Procedural | 행동 규칙 | 영구 | "날씨 질문 → weather_tool" |

---

## 8.8 메모리 구현 전략

각 메모리 유형은 서로 다른 기술로 구현한다.

### 8.8.1 Episodic Memory 구현

벡터 데이터베이스에 시간 메타데이터를 함께 저장하는 방식이 일반적이다. 각 상호작용을 임베딩하고, 타임스탬프·사용자 ID·세션 ID를 메타데이터로 추가한다. 검색 시 시간 가중치를 적용하면 최근 경험에 더 높은 우선순위를 부여할 수 있다. 6장에서 다룬 LangGraph의 Store API(`put`, `get`, `search`)가 이 패턴의 대표적인 구현이다.

### 8.8.2 Semantic Memory 구현

일화 메모리에서 패턴을 추출하여 지식 그래프에 저장한다. 엔티티와 관계를 노드·엣지로 표현하면 구조화된 지식을 질의할 수 있다. 9장에서 다루는 GraphRAG가 이 접근의 확장이다. 더 단순한 방법으로는 사용자 프로필을 JSON 문서로 유지하면서 주기적으로 갱신하는 방식이 있다.

### 8.8.3 Procedural Memory 구현

행동 규칙은 시스템 프롬프트에 명시하거나, 파인튜닝으로 모델에 내재화한다. 학습된 도구 사용 패턴은 few-shot 예제로 프롬프트에 포함할 수 있다. 비용이 허용된다면 성공적인 도구 호출 이력을 수집하여 파인튜닝 데이터셋으로 활용하는 방법도 있다.

### 8.8.4 LangGraph + 외부 저장소 조합

프로덕션 수준의 메모리 시스템은 LangGraph의 체크포인터(단기)와 Store API(장기)를 외부 저장소와 결합하여 구현한다. 6장에서 다룬 MongoDB(`langgraph-store-mongodb`)는 Episodic/Semantic 메모리에, Redis(`langgraph-checkpoint-redis`)는 Working Memory와 빠른 캐시에 적합하다. 이 조합은 메모리 유형별로 최적의 저장소를 선택하면서도, LangGraph의 통합 API로 일관되게 접근할 수 있는 장점이 있다.

---

## 8.9 RAG와 메모리: 상호보완 관계

RAG와 에이전트 메모리는 종종 혼동되지만, 그 목적이 다르다. RAG는 **외부 지식**을 검색하여 모델이 알지 못하는 정보를 제공하는 것이 목적이다. 에이전트 메모리는 **에이전트의 경험**을 축적하여 이전 상호작용의 맥락을 유지하는 것이 목적이다.

**표 8.3** RAG vs 에이전트 메모리

| 기준 | RAG | 에이전트 메모리 |
|------|-----|---------------|
| 목적 | 외부 지식 접근 | 경험 축적·활용 |
| 데이터 원천 | 문서 코퍼스 | 과거 상호작용 |
| 갱신 주기 | 문서 변경 시 | 매 상호작용마다 |
| 검색 대상 | 문서 청크 | 에피소드, 사용자 선호 |
| 대표 기술 | 벡터 DB + 임베딩 | Store API + 시간 메타데이터 |

실제 에이전트 애플리케이션은 두 접근을 결합한다. RAG로 도메인 지식을 검색하면서, Episodic Memory로 이전 질의 결과를 참조하면 중복 검색을 줄이고 응답의 일관성을 높일 수 있다. 이 장의 실습에서도 기본 RAG를 구현한 뒤, Episodic Memory를 추가하는 개선판을 구현한다.

---

## 8.10 실습: 문서 기반 Q&A 시스템

### 실습 목표

1. 문서를 청킹하고 ChromaDB에 저장한다.
2. 질문에 대해 관련 문서를 검색한다.
3. 검색 결과를 바탕으로 LLM이 답변을 생성한다.
4. 답변에 출처를 포함하여 반환한다.

### 실습 환경 설정

```bash
cd practice/chapter8
python3 -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r code/requirements.txt
cp code/.env.example code/.env  # OPENAI_API_KEY 설정
python3 code/8-6-rag-basic.py
```

### 실행 결과

**표 8.4** RAG 시스템 실행 결과

| 항목 | 값 |
|-----|-----|
| 입력 문서 수 | 3개 |
| 생성된 청크 수 | 11개 |
| 청크 크기 | 500자 |
| 오버랩 | 50자 |
| 총 실행 시간 | 22.43초 |

### 질문 및 답변 예시

질문: "asyncio.gather()와 asyncio.wait()의 차이점은 무엇인가요?"

답변: asyncio.gather()는 모든 코루틴이 완료될 때까지 기다린 후 결과를 리스트 형태로 반환한다. 반면 asyncio.wait()는 완료된 태스크와 대기 중인 태스크를 구분하여 반환하며, 더 세밀한 제어가 가능하다. [출처: asyncio_tasks.txt]

### 실습 산출물

- `practice/chapter8/data/output/ch08_qa_result.json`: Q&A 결과
- `practice/chapter8/data/output/ch08_index_stats.json`: 인덱스 통계
- `practice/chapter8/data/output/ch08_answer.txt`: 생성된 답변

---

## 8.11 프로덕션 체크리스트

프로덕션 환경에서 RAG 시스템을 운영할 때 고려해야 할 사항을 정리한다.

### 캐싱 전략

임베딩 결과를 캐싱하여 중복 계산을 방지한다. 자주 묻는 질문에 대한 답변을 캐싱하여 응답 시간과 비용을 절감한다. 문서가 업데이트되면 관련 캐시를 무효화해야 한다.

### 비용 최적화

임베딩 모델 선택이 비용에 큰 영향을 미친다. OpenAI의 text-embedding-3-small은 성능과 비용의 균형이 좋고, text-embedding-3-large는 더 높은 정확도를 제공한다. 로컬 모델(예: all-MiniLM-L6-v2)을 사용하면 비용을 크게 줄일 수 있다. 배치 처리로 API 호출을 최소화한다.

### 평가 지표

검색 품질을 평가하는 지표로 precision@k, recall@k를 사용한다. 답변 품질은 정확성, 관련성, 완전성 등으로 평가한다. 정기적인 평가와 A/B 테스트로 시스템을 개선한다.

### 모니터링

검색 실패율, 응답 시간, 사용자 피드백을 모니터링한다. 임베딩 버전을 관리하여 문서-벡터 매핑을 추적한다.

**표 8.5** RAG 프로덕션 체크리스트

| 영역 | 체크 항목 |
|-----|---------|
| 검색 | 적절한 Top-K 설정, 필터링 조건, 리랭킹 적용 |
| 비용 | 임베딩 캐싱, 배치 처리, 모델 선택 |
| 품질 | 정기적 평가, 사용자 피드백 수집 |
| 운영 | 로깅, 오류 모니터링, 버전 관리 |

---

## 8.12 실패 사례와 교훈

### 검색 실패: 관련 문서를 찾지 못함

질문과 문서의 표현 방식이 다르면 유사도가 낮아 검색에 실패한다. 쿼리 확장이나 하이브리드 검색으로 완화할 수 있다. 문서 작성 시 다양한 표현을 포함하는 것도 도움이 된다.

### 컨텍스트 과부하

너무 많은 청크를 LLM에 전달하면 컨텍스트 윈도우를 초과하거나 관련 없는 정보로 인해 답변 품질이 저하된다. 적절한 Top-K 값을 설정하고, 필요시 요약 또는 압축을 적용한다.

### 오래된 정보

문서가 업데이트되었지만 인덱스에 반영되지 않으면 오래된 정보로 답변한다. 문서 변경 감지와 자동 재인덱싱 파이프라인을 구축해야 한다.

---

## 핵심 정리

- RAG는 외부 지식을 검색하여 LLM 답변의 품질과 신뢰성을 향상시킨다.
- 인덱싱(청킹 → 임베딩 → 저장)과 검색-생성 두 단계로 구성된다.
- 에이전트 메모리는 4가지 유형으로 분류된다: Working(컨텍스트), Episodic(경험), Semantic(지식), Procedural(행동 규칙).
- RAG는 외부 지식 검색, 메모리는 에이전트 경험 축적이라는 점에서 상호보완적이다.
- LangGraph Store API + MongoDB/Redis 조합으로 프로덕션 수준의 메모리 시스템을 구현한다.
- 출처를 명시하여 답변의 검증 가능성을 높인다.

---

## 다음 장 예고

9장에서는 GraphRAG와 LightRAG를 다룬다. 벡터 검색의 한계를 극복하고, 엔티티 간 관계를 활용한 그래프 기반 검색으로 복잡한 질문에 답하는 방법을 살펴본다.

---

## 참고문헌

Eden AI. (2025). *The 2025 Guide to Retrieval-Augmented Generation (RAG)*. https://www.edenai.co/post/the-2025-guide-to-retrieval-augmented-generation-rag

Firecrawl. (2025). *Best Chunking Strategies for RAG in 2025*. https://www.firecrawl.dev/blog/best-chunking-strategies-rag-2025

LangChain. (2025). *Memory overview - LangGraph Documentation*. https://docs.langchain.com/oss/python/langgraph/memory

LangChain. (2025). *Launching Long-Term Memory Support in LangGraph*. https://www.blog.langchain.com/launching-long-term-memory-support-in-langgraph/

MongoDB. (2025). *Powering Long-Term Memory For Agents With LangGraph And MongoDB*. https://www.mongodb.com/company/blog/product-release-announcements/powering-long-term-memory-for-agents-langgraph

Redis. (2025). *LangGraph & Redis: Build smarter AI agents with memory persistence*. https://redis.io/blog/langgraph-redis-build-smarter-ai-agents-with-memory-persistence/

Weaviate. (2024). *Chunking Strategies to Improve Your RAG Performance*. https://weaviate.io/blog/chunking-strategies-for-rag
