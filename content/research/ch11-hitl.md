# 11장 리서치: Human-in-the-Loop 워크플로우

## 1. LangGraph interrupt와 Command

### interrupt 함수
- LangGraph 0.2.31 이후 권장 방식
- 그래프 실행 일시 중지, 스레드를 interrupted로 표시
- 입력을 persistence layer에 저장
- `Command(resume="response")`로 재개

### 주요 사용 사례
1. **도구 호출 검토**: 실행 전 도구 호출 승인/편집
2. **LLM 출력 검증**: 생성된 콘텐츠 검토
3. **컨텍스트 제공**: 추가 정보 명시적 요청
4. **승인 워크플로우**: API 호출, DB 변경, 금융 거래 전 일시 중지

### Command 프리미티브
- 노드 간 표현력 있는 통신 제공
- 사전 정의 엣지 없이 동적 라우팅
- 멀티에이전트 핸드오프 지원
- 상태 업데이트와 경로 결정 동시 수행

### 요구사항
- Checkpointer 필수 (상태 저장용)
- 각 단계 후 그래프 상태 체크포인트

## 2. Human-in-the-Loop Best Practices

### HITL 트리거 정의
- 위험 점수 기반 (정적 임계값 아님)
- 고영향 작업: 금융 이체, 데이터 공개, 코드 배포
- 중요 출력과 결정에 대한 사람 검토
- 엣지 케이스/이상 상황 에스컬레이션 경로

### 비동기 승인
- 저우선순위/비차단 흐름: 비동기 검토 채널 활용
- Slack, 이메일, 대시보드 라우팅
- HumanLayer 같은 프레임워크 활용

## 3. 감사 로깅 Best Practices

### 필수 로그 항목
- 누가 (actor identity): 사람, 애플리케이션, AI 에이전트
- 언제 (timestamps)
- 무엇을 (subject identity, 리소스)
- 왜 (reasoning, 불확실성 지표)

### 저장 항목
- 프롬프트
- 검색된 콘텐츠 포인터
- 도구 호출과 파라미터
- 출력
- 안전성 점수
- 승인
- 사용자 피드백

### 보안 및 보존
- 불변 로그 저장 (암호화)
- SOC 2, PCI DSS, HIPAA: 3~7년 보존
- 전송 중 및 저장 시 암호화

## 4. 거버넌스 및 보안

### 에이전트 보안
- 각 에이전트 액션 로깅
- 최소 권한 원칙 적용
- 보안 시크릿 관리
- 마이크로세그멘테이션

### 책임 추적
- AI 추천과 사람 검토, 최종 결과 함께 로깅
- AI 추론, 불확실성 지표, 최종 결정 포함
- 투명한 이벤트 체인 생성

## 5. 권장 프레임워크

- LangGraph + MCP Adapters + Permit.io: 제어, 유연성, 정책 시행
- AI Trust Layer: 그룹별 권한, PII 삭제, 감사 로그, 스로틀링
- 고위험 결정에 대한 에스컬레이션으로 HITL 유지
- 최종 승인은 항상 사람 검토자에게

## 참고 자료

- [LangGraph Human-in-the-Loop Concepts](https://langchain-ai.github.io/langgraphjs/concepts/human_in_the_loop/)
- [LangGraph Interrupts Documentation](https://docs.langchain.com/oss/python/langgraph/interrupts)
- [LangChain Blog - interrupt for HITL](https://blog.langchain.com/making-it-easier-to-build-human-in-the-loop-agents-with-interrupt/)
- [Permit.io - HITL Best Practices](https://www.permit.io/blog/human-in-the-loop-for-ai-agents-best-practices-frameworks-use-cases-and-demo)
- [Latitude - Audit Logs in AI Systems](https://latitude-blog.ghost.io/blog/audit-logs-in-ai-systems-what-to-track-and-why/)
- [ISACA - Safeguarding Agentic AI Workflows](https://www.isaca.org/resources/news-and-trends/industry-news/2025/safeguarding-the-enterprise-ai-evolution-best-practices-for-agentic-ai-workflows)
