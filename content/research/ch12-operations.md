# 12장 리서치: 배포·관측·비용 최적화

## 1. LLM Observability

### 정의
- LLM 앱의 추적, 모니터링, 평가 관행
- 비결정적 특성으로 디버깅/최적화가 어려움

### 2025 트렌드
- 멀티스텝 에이전트 워크플로우(LangGraph, AutoGen) 지원
- 중첩 span을 통한 깊은 에이전트 추적

### Langfuse
- 가장 많이 사용되는 오픈소스 LLM 관측 도구
- 포괄적인 트레이싱, 평가, 프롬프트 관리, 메트릭
- 모델/프레임워크 무관
- 셀프호스팅 옵션 제공
- OpenTelemetry, LangChain, OpenAI SDK 통합
- SDK v3: OTEL 네이티브, 토큰 사용량, 비용 추적, 프롬프트 링킹

### LangSmith
- LangChain 팀의 관리형 관측 스위트
- 대시보드, 알림, HITL 평가
- LangChain/LangGraph 생태계와 심층 통합
- 셀프호스팅: Enterprise 전용
- 무료 티어: 월 5K 트레이스

### 비교

| 기능 | Langfuse | LangSmith |
|-----|----------|-----------|
| 셀프호스팅 | MIT 라이선스, 쉬움 | Enterprise only |
| 적합 대상 | 오픈소스 팀 | LangChain 사용자 |
| 요금 | Hobby 무료 (50K units) | 시트 기반 |

### 핵심 기능
- OpenTelemetry 지원
- 토큰 및 비용 모니터링
- 프롬프트 및 평가 통합
- 협업 도구
- 필터링 및 검색

## 2. API 가격 (2025)

### OpenAI GPT-4o
- 입력: $5.00/1M 토큰 (캐시: $2.50)
- 출력: $20.00/1M 토큰
- 16개월간 83% 가격 인하

### OpenAI GPT-4o-mini
- 입력: $0.60/1M 토큰 (캐시: $0.30)
- 출력: $2.40/1M 토큰
- GPT-4o 대비 94% 저렴

### Anthropic Claude Sonnet 4
- 입력: $3.00/1M 토큰
- 출력: $15.00/1M 토큰
- GPT-4와 경쟁력 있는 가격

### Anthropic Claude Haiku
- 입력: ~$0.25/1M 토큰
- 출력: ~$1.25/1M 토큰

## 3. 비용 최적화 전략

### 프롬프트 최적화
- 시스템 프롬프트 압축: 62% 크기 감소
- 컨텍스트 프루닝: 41% 토큰 사용 감소
- 응답 길이 제어: 33% 출력 비용 절감

### 캐싱
- Anthropic: 캐시 쓰기 1.25× 기본 입력
- 캐시 읽기: 0.1× 기본 (90% 절감)

### 하이브리드 아키텍처
- 오픈소스 모델로 초기 처리
- GPT-4o로 정제
- 63% 토큰 비용 감소

### 모델 계층화
- 비용 민감 작업: Haiku/Mini 사용
- 중요 쿼리: Opus/GPT-4o 예약
- 60-80% 비용 절감 가능

## 참고 자료

- [Langfuse - LLM Observability](https://langfuse.com/docs/observability/overview)
- [LakeFS - LLM Observability Tools 2026](https://lakefs.io/blog/llm-observability-tools/)
- [OpenAI API Pricing](https://platform.openai.com/docs/pricing)
- [IntuitionLabs - LLM API Pricing Comparison 2025](https://intuitionlabs.ai/articles/llm-api-pricing-comparison-2025)
- [LaoZhang - GPT-4o Pricing Guide](https://blog.laozhang.ai/ai-tools/openai-gpt4o-pricing-guide/)
