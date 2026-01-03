# 8장 리서치 결과: RAG의 기본과 프로덕션 체크리스트

## 조사일: 2026-01-02

## 1. RAG 개요

### 정의
RAG(Retrieval Augmented Generation)는 검색 메커니즘과 생성 모델을 통합하는 하이브리드 프레임워크다. 외부 데이터를 검색하여 LLM의 응답 품질을 향상시킨다.

### 2025년 현황
- 2025년 기준 프로덕션 AI 애플리케이션의 약 60%가 RAG를 사용
- 고객 지원 챗봇, 내부 지식 베이스 등에 광범위하게 적용

### RAG vs 파인튜닝
| 기준 | RAG | 파인튜닝 |
|-----|-----|---------|
| 지식 업데이트 | 문서 추가로 즉시 | 재학습 필요 |
| 비용 | 임베딩/검색 비용 | 학습 비용 높음 |
| 출처 추적 | 가능 | 불가능 |
| 환각 감소 | 근거 기반 답변 | 제한적 |

## 2. RAG 아키텍처 발전

### 기본 (Monolithic) RAG
- 단순한 검색-생성 파이프라인
- 반복적이고 직관적인 쿼리에 적합
- 예: "휴가 정책이 뭔가요?"

### Two-Step Query Rewriting
- 사용자 입력이 모호할 때 사용
- 첫 번째 LLM이 쿼리를 정제하여 검색 품질 향상

### Agentic RAG
- 복잡한 추론, 워크플로우 실행, 도구 사용이 필요할 때
- Self-Reflective 메커니즘으로 검색 필요성 동적 판단

### Long RAG
- 긴 문서 처리에 최적화
- 섹션 또는 전체 문서 단위로 검색
- 컨텍스트 보존, 계산 비용 절감

## 3. 청킹 전략

### 청킹 유형
1. **고정 크기 청킹**: 일정 토큰/문자 수로 분할
2. **의미 기반 청킹**: 의미적 유사성 기반 분할
3. **재귀적 청킹**: RecursiveCharacterTextSplitter (권장 시작점)
4. **Agentic 청킹**: AI가 동적으로 분할 방식 결정

### 최적 설정
- 청크 크기: 256-512 토큰
- 오버랩: 10-20% (500 토큰 청크 시 50-100 토큰)

### 성능 비교
- Semantic Chunking: 70% 정확도 향상 (compute 비용 높음)
- Page-level Chunking: NVIDIA 벤치마크 0.648 정확도

### 트레이드오프
- 작은 청크: 쿼리 매칭 정확 but 컨텍스트 손실
- 큰 청크: 관계 보존 but 임베딩 희석

## 4. ChromaDB와 LangChain

### ChromaDB 특징
- 오픈소스 벡터 데이터베이스
- 고차원 임베딩 저장, 인덱싱, 검색
- RAG 워크플로우, 시맨틱 검색에 최적화

### LangChain 통합
- 파이프라인 오케스트레이션: load → split → embed → store → retrieve → generate
- 유틸리티: RecursiveCharacterTextSplitter, RetrievalQA, MultiQueryRetriever

### 임베딩 모델
- OpenAI Embeddings: 고품질, 비용 발생
- HuggingFace all-MiniLM-L6-v2: 경량, 로컬 실행 가능

## 5. 프로덕션 고려사항

### 일반적인 문제
- 검색 실패: 관련 문서 미검색 → 튜닝 필요
- 컨텍스트 과부하: 검색 문서가 LLM 윈도우 초과 → 요약/압축 필요
- 신뢰 문제: 나쁜 RAG는 RAG 없는 것보다 나쁨

### 평가 지표
- precision@k, recall@k
- 생성 품질 메트릭
- 벤치마크: FRAMES, LONG2RAG

### 운영 베스트 프랙티스
- 임베딩 버전 관리
- VectorDB 마이그레이션 계획
- 모듈형 설계 (검색 모듈 교체 가능)

## 참고 자료

- Eden AI. (2025). *The 2025 Guide to Retrieval-Augmented Generation (RAG)*. https://www.edenai.co/post/the-2025-guide-to-retrieval-augmented-generation-rag
- Towards Data Science. (2024). *Six Lessons Learned Building RAG Systems in Production*. https://towardsdatascience.com/six-lessons-learned-building-rag-systems-in-production/
- Firecrawl. (2025). *Best Chunking Strategies for RAG in 2025*. https://www.firecrawl.dev/blog/best-chunking-strategies-rag-2025
- Weaviate. (2024). *Chunking Strategies to Improve Your RAG Performance*. https://weaviate.io/blog/chunking-strategies-for-rag
- DataCamp. (2025). *Chunking Strategies for AI and RAG Applications*. https://www.datacamp.com/blog/chunking-strategies
