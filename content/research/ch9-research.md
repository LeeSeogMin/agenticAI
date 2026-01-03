# 9장 리서치 결과: GraphRAG/LightRAG/KAG

## 조사일: 2026-01-02

## 1. 벡터 RAG의 한계

### 문제점
- 단순 유사도 기반 검색으로 관계 추론 불가
- 다중 홉(multi-hop) 질문 처리 어려움
- "데이터의 상위 5개 테마는?" 같은 전역적 질문에 취약
- 엔티티 간 관계 파악 불가

### GraphRAG가 필요한 이유
- 벡터 검색은 쿼리를 올바른 정보로 안내할 방향성이 없음
- LLM 생성 지식 그래프 구조가 데이터셋 전체의 주제를 파악

## 2. Microsoft GraphRAG

### 개요
- 구조화된 계층적 RAG 접근법
- 원시 텍스트에서 지식 그래프 추출
- 커뮤니티 계층 구축 및 요약 생성
- 2024년 GitHub 공개, 2025년 2월 논문 업데이트

### 아키텍처
1. **인덱싱**: 소스 문서에서 엔티티 지식 그래프 추출
2. **커뮤니티 요약**: 관련 엔티티 그룹의 요약 사전 생성
3. **쿼리 처리**: 각 커뮤니티 요약으로 부분 응답 생성 후 최종 통합

### 검색 전략
- **글로벌 검색**: 데이터셋 전체에 대한 질문
- **로컬 검색**: 특정 엔티티/관계에 대한 질문

### 장점
- 100만 토큰 범위 데이터셋에서 포괄성과 다양성 크게 향상
- 데이터셋의 의미적 구조 사전 파악 가능

### 단점
- 인덱싱 비용 높음
- 아키텍처 복잡성
- 그래프 구축/유지 비용

## 3. LightRAG (HKUDS)

### 개요
- EMNLP2025 발표: "Simple and Fast Retrieval-Augmented Generation"
- 홍콩대학교 데이터과학 연구팀 개발
- GraphRAG의 경량화 버전

### 핵심 특징
- **듀얼 레벨 검색**: 로컬(상세) + 글로벌(거시) 통합
- **모듈형 설계**: 문서 파싱, 인덱스 구축, 검색, 생성 분리
- **지식 그래프 + 벡터 검색 결합**

### 성능
- 쿼리 지연 ~30% 감소 (~80ms vs 표준 RAG ~120ms)
- 벡터 DB 조회와 경량 그래프로 효율성 달성
- Naive RAG와 GraphRAG 대비 벤치마크 우수

### 2025년 업데이트
- 9월: 오픈소스 LLM(Qwen3-30B-A3B) 지식 그래프 추출 정확도 향상
- 8월: Reranker 지원, 문서 삭제 시 자동 KG 재생성
- 6월: RAG-Anything 출시 (멀티모달 RAG)

### 저장소 지원
- Key-Value: Json, PostgreSQL, Redis
- Vector: FAISS, Chroma, Milvus

### GraphRAG vs LightRAG

| 기준 | GraphRAG | LightRAG |
|-----|----------|----------|
| 개발사 | Microsoft | HKUDS |
| 인덱싱 비용 | 높음 | 낮음 |
| 검색 전략 | 글로벌/로컬 분리 | 듀얼 레벨 통합 |
| 증분 업데이트 | 제한적 | 지원 |
| 적합 사용 | 대규모 분석 | 빠른 프로토타입 |

## 4. KAG (Knowledge Augmented Generation)

### 개요
- OpenSPG 엔진과 LLM 기반 논리적 추론/검색 프레임워크
- 전문 도메인 지식베이스용 Q&A 솔루션
- RAG 벡터 유사도 계산의 모호성 극복

### 핵심 특징
- **도메인 특화 지식 그래프**: 일반 KG가 아닌 전문 분야별 구축
- **다중 홉 추론**: 정보 연결 및 합성
- **DIKW 모델 기반 계층화**: Data-Information-Knowledge-Wisdom
- **상호 인덱싱**: 텍스트 청크와 KG 노드 연결

### 2025년 업데이트
- "Lightweight Build" 모드: 지식 구축 토큰 비용 89% 절감
- 도메인 지식 주입, 스키마 커스터마이징 지원
- 시각적 쿼리 분석 기능

### GraphRAG vs KAG

| 측면 | GraphRAG | KAG |
|-----|----------|-----|
| KG 유형 | 일반 지식 그래프 | 도메인 특화 지식 그래프 |
| 추론 능력 | 사실 검색, 복잡 쿼리 취약 | 다중 홉 추론으로 정밀 답변 |
| 노이즈 처리 | OpenIE 오류로 노이즈 발생 가능 | 오류 감소, 정밀도 향상 |

## 참고 자료

- Microsoft GraphRAG: https://github.com/microsoft/graphrag
- GraphRAG Documentation: https://microsoft.github.io/graphrag/
- LightRAG: https://github.com/HKUDS/LightRAG
- KAG: https://github.com/OpenSPG/KAG
- GraphRAG Paper: https://arxiv.org/abs/2404.16130
