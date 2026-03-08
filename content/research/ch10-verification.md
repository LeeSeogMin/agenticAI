# 10장 리서치: 신뢰성과 검증 파이프라인

## 1. LLM 환각 탐지 및 방지 기술 (2025)

### 환각 재정의
- OpenAI 2025년 논문: next-token 학습 목표와 리더보드가 보정된 불확실성보다 자신감 있는 추측에 보상
- 목표 전환: "제로 환각" 추구 → "측정 가능하고 예측 가능한 불확실성 관리"

### 탐지 기법
1. **Semantic Entropy** (Nature 발표): 단어 시퀀스가 아닌 의미 수준에서 불확실성 계산
2. **LLM Prompt-based Detection**: 정확도 75% 이상, 상대적 저비용
3. **LLM-as-a-Judge**: 프롬프트 엔지니어링 + 다단계 추론 결합

### 완화 전략
- **RAG**: 실시간/도메인 특화 지식 처리 강화
- **Training & Alignment**: 보정 인식 보상, 불확실성 친화적 평가 메트릭
- **Prompt Engineering**: 작은 문구 변화도 환각 가능성에 영향

## 2. Chain-of-Verification (CoVe)

### 4단계 프로세스
1. 초기 응답 작성 (Draft)
2. 검증 질문 계획 (Plan)
3. 독립적으로 검증 질문 답변 (Verify)
4. 최종 검증된 응답 생성 (Final)

### 변형
- **Joint**: 단일 프롬프트로 검증 질문+답변 (오류 반복 가능성)
- **Factor + Revise**: 각 검증 답변 독립 검증 후 원본 수정 (권장)

### 성과
- LLaMA-65B: FactScore 63.7 → 71.4 상승
- 기술 문서/코드 리뷰에서 40% 정확도 향상
- CoV-RAG: RAG 파이프라인에 검증 모듈로 통합

## 3. RAG Faithfulness 평가

### 핵심 메트릭
- **Answer Relevancy**: 응답이 입력에 얼마나 관련 있는가
- **Faithfulness**: 응답이 검색 컨텍스트에 대해 환각을 포함하는가
- **Contextual Relevancy**: 검색 컨텍스트가 입력에 얼마나 관련 있는가
- **Groundedness**: RAG 답변이 검색 문서에 의해 지원되는 정도

### NLI 기반 검증
- SummaC: 문서-요약 문장 쌍 간 NLI entailment 점수 집계
- 초기 탐지 방법은 NLI/QA 시스템에 의존

### Correctness vs Faithfulness
- Citation이 정확해도 모델이 실제로 문서를 사용하지 않았다면 unfaithful
- Command-R+: 내부 지식으로 답변 생성 후 post-hoc citation 수행

### 2025 Best Practice
- Ragas, TruLens, DeepEval 등 오픈소스 프레임워크 활용
- 오프라인 테스트, 노드 레벨 평가, 자동화된 로그 평가, CI/CD 게이트

## 4. LangGraph Human-in-the-Loop

### 핵심 함수
- `interrupt`: 그래프 실행 일시 중지, 사람에게 정보 제시
- `Command`: 사람 입력으로 상태 업데이트, 다음 노드로 흐름 제어

### 3가지 주요 액션
1. **Approve or Reject**: 중요 단계 전 일시 중지, 승인/거부
2. **Edit Graph State**: 상태 검토 및 편집
3. **Get Input**: 특정 단계에서 명시적 입력 요청

### 작동 방식
```python
from langgraph.types import interrupt, Command

def human_approval(state):
    is_approved = interrupt({"question": "Is this correct?"})
    if is_approved:
        return Command(goto="proceed")
    else:
        return Command(goto="abort")
```

## 5. Self-Consistency Sampling

### 개념
- 동일 프롬프트를 여러 번 실행, 다수결로 최종 답변 선택
- Chain-of-Thought와 결합 시 더 강력

### Temperature 설정
- Self-consistency는 높은 temperature (T > 0) 사용
- T = 1.2가 Self-Consistency에 최적
- TURN: 자동 temperature 최적화 접근법

### 개선 방법
- **CISC (Confidence-Informed SC)**: 신뢰도 점수 기반 가중 다수결
- **RASC**: 출력과 근거를 동적으로 평가하여 샘플링 효율성 향상

## 참고 자료

- [Lakera - LLM Hallucinations Guide](https://www.lakera.ai/blog/guide-to-hallucinations-in-large-language-models)
- [Deepchecks - LLM Hallucination Detection](https://www.deepchecks.com/llm-hallucination-detection-and-mitigation-best-techniques/)
- [Nature - Semantic Entropy](https://www.nature.com/articles/s41586-024-07421-0)
- [arXiv - Chain-of-Verification](https://arxiv.org/abs/2309.11495)
- [Confident AI - RAG Evaluation Metrics](https://www.confident-ai.com/blog/rag-evaluation-metrics-answer-relevancy-faithfulness-and-more)
- [LangChain - Human-in-the-Loop](https://docs.langchain.com/oss/python/langchain/human-in-the-loop)
- [Learn Prompting - Self-Consistency](https://learnprompting.org/docs/intermediate/self_consistency)
