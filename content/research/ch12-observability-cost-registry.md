# 12장 리서치: 관측성 플랫폼, 비용 최적화, MCP 레지스트리

> 조사일: 2026-02-11
> 범위: 관측성(Observability) 플랫폼 비교, AI 에이전트 비용 최적화 전략, MCP 레지스트리 및 서버 디스커버리

---

## 1. 관측성 플랫폼 비교 (2025-2026)

### 1.1 비교 요약표

| 플랫폼 | 무료 티어 | 유료 플랜 | 배포 모델 | 오픈소스 | 핵심 강점 | 최적 대상 |
|--------|----------|----------|----------|---------|----------|----------|
| **LangSmith** | 5,000 트레이스/월, 1석 | Plus $39/사용자/월 (10K 트레이스 포함) | 클라우드, BYOC, 자체 호스팅 | 비공개 | LangChain 네이티브 통합, 평가 프레임워크 | LangChain/LangGraph 사용 팀 |
| **Arize Phoenix** | 오픈소스 무제한 (자체 호스팅) | 클라우드 무료 인스턴스, 엔터프라이즈 AX $50K-100K/년 | 자체 호스팅(Docker), 클라우드 | Apache 2.0 | OpenTelemetry 기반, 에이전트 평가 심층 지원 | 기존 관측 스택 보유 엔터프라이즈 |
| **Langfuse** | 클라우드: 1M 스팬/월, 무제한 사용자 | Pro $249/월 | 클라우드, 자체 호스팅(Docker/K8s) | MIT | 50+ 프레임워크 통합, 완전 자체 호스팅 가능 | 데이터 주권 중시 팀, 스타트업 |
| **Braintrust** | 1M 스팬, 10K 스코어, 14일 보존 | Pro $249/월 (무제한 스팬, 1개월 보존) | 클라우드 | 비공개 | 평가-중심 워크플로우, AI 프록시, Brainstore | 평가/실험 중심 ML 팀 |
| **Helicone** | 100K 요청/월, 5석 | Pro $25/월 (무제한 요청) | 클라우드, 자체 호스팅 | 오픈소스 | 비용 추적 특화, AI 게이트웨이, 보안 내장 | 비용 관리 우선 팀, 게이트웨이 필요 |

### 1.2 LangSmith

LangSmith는 LangChain이 개발한 AI 에이전트 관측성(Observability) 및 평가(Evaluation) 플랫폼이다. LangChain과 LangGraph로 구축된 에이전트에 네이티브로 통합되며, 외부 프레임워크도 지원한다.

**핵심 기능:**
- **트레이싱(Tracing)**: 에이전트 행동에 대한 완전한 가시성 제공. LLM 호출, 도구 실행, 검색 단계를 포함하는 구조화된 트레이스(Trace) 기록
- **실시간 모니터링**: 토큰 사용량, 지연시간(P50, P99), 에러율, 비용 분석, 피드백 점수를 추적하는 커스텀 대시보드
- **평가 프레임워크**: 프로덕션 트레이스를 평가 케이스로 변환, CI/CD 파이프라인에서 회귀 테스트(Regression Test) 실행
- **비용 추적(Cost Tracking)**: 모델별, 프로젝트별 비용 분석

**가격 정책:**
- Developer (무료): 1석, 5,000 기본 트레이스/월
- Plus: $39/사용자/월, 10,000 기본 트레이스 포함
- Enterprise: 맞춤 가격
- 추가 트레이스: 기본 트레이스 $2.50/1,000건 (14일 보존), 확장 트레이스 $5.00/1,000건 (400일 보존)
- 배포 옵션: 관리형 클라우드, BYOC(Bring Your Own Cloud), 자체 호스팅

> 출처: https://www.langchain.com/pricing, https://www.langchain.com/langsmith/observability

### 1.3 Arize Phoenix

Arize AI가 개발한 오픈소스 AI 관측성 플랫폼으로, OpenTelemetry와 OpenInference 계측(Instrumentation)을 기반으로 구축되었다. GitHub 스타 7,800개 이상을 보유하고 있다.

**핵심 기능:**
- **분산 트레이싱(Distributed Tracing)**: 에이전트 루프 전체 가시성 - LLM 호출, 도구 실행, 검색(Retrieval), 추론 단계
- **평가 프레임워크**: 내장 LLM 평가자(Evaluator) + Ragas, DeepEval, Cleanlab 통합
- **프롬프트 관리(Prompt Management)**: 2025년 4월 출시. 모델과 워크플로우 간 프롬프트 템플릿 생성, 버전 관리, 재사용
- **프롬프트 플레이그라운드(Prompt Playground)**: 프롬프트 병렬 실험, 수정된 입력으로 스팬(Span) 재실행
- **비용 추적**: 2025년 6월 출시. 모델, 프롬프트, 사용자별 LLM 사용량 및 비용 추적
- **데이터셋 및 실험**: 트레이스에서 데이터셋 생성, 체계적 A/B 실험 실행

**가격 정책:**
- 오픈소스 (자체 호스팅): 무료, 기능 제한 없음
- 클라우드 무료 인스턴스: app.phoenix.arize.com
- 클라우드 관리형 인프라: $50-500/월
- Arize AX (엔터프라이즈): $50K-100K/년 (컴플라이언스, 코파일럿 등 포함)

> 출처: https://github.com/Arize-ai/phoenix, https://phoenix.arize.com/pricing/, https://arize.com/docs/phoenix

### 1.4 Langfuse

Langfuse는 오픈소스(MIT 라이선스) LLM 엔지니어링 플랫폼으로, Y Combinator W23 배치 출신이다. 자체 호스팅에 강점을 가진다.

**핵심 기능:**
- **구조화된 로깅**: 정확한 프롬프트, 모델 응답, 토큰 사용량, 지연시간, 도구/검색 단계를 포함하는 모든 요청의 구조화된 로그 캡처
- **다양한 통합**: Python/JS 네이티브 SDK, 50개 이상의 라이브러리/프레임워크 통합(LangChain, OpenAI SDK, LiteLLM 등), OpenTelemetry 지원
- **세션 추적**: 멀티턴 대화를 세션(Session)으로 추적
- **프롬프트 관리**: 프롬프트 버전 관리 및 배포
- **평가(Evals)**: 자동화된 평가 실행, 데이터셋 관리
- **분석 대시보드**: 메트릭 개요, 비용 분석, 모델별 가격 티어 추적 (2025년 12월 출시)

**자체 호스팅 옵션:**
- Docker Compose: 소규모 배포, 단일 VM에서 5분 내 실행 가능
- Kubernetes (Helm): 프로덕션 환경 권장, 스케일링 지원
- VPC 또는 온프레미스: 고보안 환경, 인터넷 접속 선택 사항

**가격 정책:**
- 자체 호스팅 (FOSS): 무료, 기능 제한 없음, 이벤트/트레이스 수 제한 없음
- 클라우드 Hobby: 1M 트레이스 스팬/월, 무제한 사용자, 10,000 평가 실행 (무료)
- 클라우드 Pro: $249/월부터
- Enterprise: 맞춤 가격 (SSO, SOC2, ISO27001, 프로젝트 레벨 RBAC)

> 출처: https://langfuse.com/docs/observability/overview, https://langfuse.com/pricing, https://langfuse.com/self-hosting, https://github.com/langfuse/langfuse

### 1.5 Braintrust

Braintrust는 평가(Evaluation) 중심의 AI 관측성 플랫폼으로, 프로덕션 트레이스를 평가 케이스로 원클릭 변환하는 워크플로우가 특징이다.

**핵심 기능:**
- **평가(Evals)**: 실제 데이터로 AI 테스트, 변경사항이 성능을 개선하는지 저하시키는지 판별
- **프로덕션 모니터링**: 실시간 모델 응답 추적, 품질 저하 시 알림
- **AI 프록시(AI Proxy)**: OpenAI 호환 단일 API로 OpenAI, Anthropic, Google 등 여러 모델 제공. 모든 호출 자동 트레이싱 및 캐싱
- **Loop**: 프롬프트, 스코어러(Scorer), 데이터셋 작성 및 최적화를 자동화하는 내장 에이전트
- **Brainstore**: AI 애플리케이션 로그/트레이스 전용 스토리지. 기존 대비 24배 빠른 쿼리·필터·분석

**가격 정책:**
- Free: 1M 스팬, 10K 스코어, 14일 보존 (5사용자)
- Pro: $249/월, 무제한 스팬, 5GB 데이터, 1개월 보존
- Enterprise: 맞춤 가격

> 출처: https://www.braintrust.dev/pricing, https://www.braintrust.dev/

### 1.6 Helicone

Helicone은 비용 추적(Cost Tracking)에 특화된 오픈소스 LLM 관측성 플랫폼이자 AI 게이트웨이(AI Gateway)이다. Y Combinator W23 배치 출신이다.

**핵심 기능:**
- **비용 추적 및 최적화**: Model Registry v2 시스템으로 300개 이상 모델의 비용 정밀 계산. 비용 기반 라우팅(Cost-Based Routing)으로 최저가 공급자 자동 선택. 스마트 폴백(Smart Fallback)으로 장애 시 차순위 저비용 옵션으로 라우팅
- **비용 알림**: 지출 임계값 설정, 실시간 알림. 점진적 임계값(50%, 80%, 95%) 및 개발/프로덕션 환경별 한도 설정
- **세션(Session) 추적**: 사용자 여정을 다중 상호작용에 걸쳐 추적
- **보안**: Meta의 Prompt Guard 모델로 프롬프트 인젝션(Prompt Injection) 탐지, Llama Guard로 고급 보호

**가격 정책:**
- Free: $0/월, 100K 요청, 5석, 모니터링 및 대시보드
- Pro: $25/월, 무제한 요청, 버킷 캐싱, 사용자 관리, GraphQL API, 10석, 2GB 스토리지
- Priority Support: $349/월부터, 전용 지원 채널, Helm 차트, 기능 요청 우선순위
- Custom: SOC-2 컴플라이언스, 자체 배포 관리, 커스텀 ETL
- 자체 호스팅: 무료 (오픈소스)

> 출처: https://www.helicone.ai/pricing, https://github.com/Helicone/helicone, https://docs.helicone.ai/guides/cookbooks/cost-tracking

### 1.7 플랫폼 선택 가이드

| 상황 | 권장 플랫폼 | 이유 |
|------|-----------|------|
| LangChain/LangGraph 중심 개발 | LangSmith | 네이티브 통합, 평가 파이프라인 |
| 오픈소스 + 자체 호스팅 필수 | Langfuse 또는 Arize Phoenix | MIT/Apache 2.0, 완전한 기능 무료 |
| 기존 OpenTelemetry 스택 활용 | Arize Phoenix | OTLP 프로토콜 네이티브 지원 |
| 비용 관리가 최우선 과제 | Helicone | 비용 추적 특화, AI 게이트웨이 내장 |
| 평가/실험 중심 워크플로우 | Braintrust | 트레이스→평가 원클릭 변환, AI 프록시 |
| 최소 비용으로 시작 | Helicone Free (100K 요청) 또는 Langfuse Hobby (1M 스팬) | 넉넉한 무료 티어 |

---

## 2. AI 에이전트 비용 최적화

### 2.1 모델 라우팅 (Model Routing)

모델 라우팅은 작업 복잡도에 따라 적절한 모델을 자동 선택하여 비용을 최적화하는 전략이다. 핵심 원칙은 "문제를 풀 수 있는 가장 저렴한 모델에 쿼리를 보내는 것"이다.

**모델 티어별 가격 계층 (2026년 기준, Anthropic Claude):**

| 모델 | 입력 토큰 (MTok당) | 출력 토큰 (MTok당) | 용도 |
|------|-------------------|-------------------|------|
| Claude Haiku 4.5 | $1 | $5 | 단순 분류, 요약, 라우팅 판단 |
| Claude Sonnet 4.5 | $3 | $15 | 복잡한 추론, 코딩, 에이전트 태스크 |
| Claude Opus 4.6 | $5 | $25 | 최고 수준 추론, 심층 분석 |

> 출처: https://platform.claude.com/docs/en/about-claude/pricing

**비교 참고 (타사 모델, 2026년 초 기준):**

| 모델 | 입력 토큰 (MTok당) | 출력 토큰 (MTok당) |
|------|-------------------|-------------------|
| GPT-4o (OpenAI) | $2.50 | $10.00 |
| Gemini 2.0 Flash (Google) | $0.10 | $0.40 |
| Gemini Flash-Lite (Google) | $0.075 | $0.30 |

**라우팅 전략 예시:**

```
사용자 쿼리 → [라우터 (Haiku 4.5)]
    ├─ 단순 질문 (분류, FAQ)    → Haiku 4.5   ($1/MTok 입력)
    ├─ 중간 복잡도 (코드 생성)   → Sonnet 4.5  ($3/MTok 입력)
    └─ 고난도 추론 (수학, 분석)  → Opus 4.6    ($5/MTok 입력)
```

**비용 절감 효과:**
- 스마트 모델 라우팅만으로 50% 이상의 비용 절감 달성 가능
- 프롬프트 캐싱, 모델 라우팅, 인프라 최적화를 조합하면 70% 이상 비용 절감 실현
- 주의사항: 저렴한 모델 사용 시, 고가 모델로 결과를 검증(Verification)하는 추가 비용이 발생할 수 있음

> 출처: https://www.aipricingmaster.com/blog/10-AI-Cost-Optimization-Strategies-for-2026, https://www.agentframeworkhub.com/blog/ai-agent-production-costs-2026

### 2.2 적응형 추론 제어 (Adaptive Reasoning Control)

Anthropic Claude는 추론 깊이(Reasoning Depth)를 제어하는 두 가지 메커니즘을 제공한다.

#### 2.2.1 노력 매개변수 (Effort Parameter)

Claude Opus 4.6에서 권장되는 방식으로, `budget_tokens`를 대체한다. 단일 매개변수로 텍스트 응답, 도구 호출, 확장 사고(Extended Thinking) 모두에 영향을 미친다.

**노력 수준:**

| 수준 | 설명 | 적합한 용도 |
|------|------|-----------|
| `max` | 제약 없는 최대 역량. Opus 4.6 전용 | 가장 깊은 추론과 철저한 분석이 필요한 태스크 |
| `high` | 높은 역량. 매개변수 미설정 시 기본값 | 복잡한 추론, 난이도 높은 코딩, 에이전트 태스크 |
| `medium` | 균형 잡힌 접근. 적당한 토큰 절약 | 속도, 비용, 성능 균형이 필요한 에이전트 태스크 |
| `low` | 가장 효율적. 상당한 토큰 절약, 일부 역량 감소 | 단순 태스크의 최고 속도와 최저 비용, 서브에이전트 |

**사용법:**

```python
response = client.messages.create(
    model="claude-opus-4-6",
    max_tokens=4096,
    messages=[{"role": "user", "content": "간단한 질문..."}],
    output_config={
        "effort": "low"  # 토큰 절약 모드
    }
)
```

**도구 사용과의 상호작용:**
- 낮은 노력 수준: 더 적은 도구 호출, 여러 작업을 단일 호출로 결합, 전문(Preamble) 없이 바로 실행
- 높은 노력 수준: 더 많은 도구 호출, 실행 전 계획 설명, 상세한 변경 요약

> 출처: https://platform.claude.com/docs/en/build-with-claude/effort

#### 2.2.2 확장 사고와 예산 토큰 (Extended Thinking & Budget Tokens)

`budget_tokens`는 Claude가 내부 추론 과정에 사용할 수 있는 최대 토큰 수를 결정한다. 최소 예산은 1,024토큰이며, 최소값에서 시작하여 점진적으로 증가시키는 것이 권장된다.

**적용 지침:**
- Claude Opus 4.6: `budget_tokens` 대신 적응형 사고(Adaptive Thinking) + 노력 매개변수 사용 권장. `budget_tokens`는 여전히 사용 가능하나 향후 제거 예정
- Claude Opus 4.5: 수동 사고(Manual Thinking) 모드에서 `budget_tokens`와 노력 매개변수 병용
- 하이브리드 접근: 기본적으로 사고 모드를 끄고, 복잡도 기준을 충족하는 특정 턴에서만 활성화하여 지연시간과 비용을 최적화

> 출처: https://platform.claude.com/docs/en/build-with-claude/extended-thinking, https://platform.claude.com/docs/en/build-with-claude/adaptive-thinking

### 2.3 토큰 효율적 도구 호출 (Token-Efficient Tool Calling)

#### 2.3.1 토큰 효율적 도구 사용 (Token-Efficient Tool Use)

Anthropic이 2025년 2월에 도입한 기능으로, 출력 토큰 소비를 최대 70% 줄인다. 초기 사용자 평균 14% 감소를 확인하였다.

**사용법:**
```
헤더: anthropic-beta: token-efficient-tools-2025-02-19
```

> 출처: https://www.anthropic.com/news/token-saving-updates

#### 2.3.2 프로그래밍 방식 도구 호출 (Programmatic Tool Calling, PTC)

2025년 11월 공개 베타로 출시된 기능이다. Claude가 코드 실행 내에서 도구를 호출하여 지연시간과 토큰 사용량을 줄인다.

**성능 향상:**
- 복잡한 리서치 태스크에서 평균 43,588 → 27,297 토큰으로 **37% 감소**
- Claude가 단일 코드 블록에서 20개 이상의 도구 호출을 조율할 때, 19번 이상의 추론 패스(Inference Pass)를 제거
- 프로그래밍 방식 호출의 도구 결과는 입력/출력 토큰 사용량에 포함되지 않음 - 최종 코드 실행 결과와 Claude의 응답만 계산

**사용법:**
```
헤더: anthropic-beta: advanced-tool-use-2025-11-20
코드 실행 도구(Code Execution Tool) 활성화 필요
```

> 출처: https://www.anthropic.com/engineering/advanced-tool-use, https://platform.claude.com/docs/en/agents-and-tools/tool-use/programmatic-tool-calling

#### 2.3.3 도구 검색 (Tool Search) - Claude Code 사례 연구

Claude Code의 Tool Search 기능은 MCP 도구 정의가 10K 토큰을 초과할 때 동적 도구 디스커버리(Dynamic Tool Discovery)를 활성화한다. 모든 도구 정의를 컨텍스트에 미리 로드하는 대신, 필요한 도구만 검색하여 로드한다.

**동작 메커니즘:**
1. **탐지(Detection)**: MCP 도구 설명이 10K 토큰을 초과하는지 감지
2. **지연 로딩(Deferral)**: `defer_loading: true`로 도구를 표시
3. **온디맨드 디스커버리(On-demand Discovery)**: 쿼리당 3-5개의 관련 도구만 로드 (약 3K 토큰)

**토큰 절약 효과:**
- Tool Search 자체 오버헤드: 약 500토큰
- 컨텍스트 오염(Context Pollution) 46.9% 감소: 51K 토큰 → 8.5K 토큰으로 축소
- 전체 토큰 85% 감소 사례: 일부 워크플로우에서 110,473 → 15,919 총 토큰 (85.6% 감소)
- Claude Code에서 MCP 서버 사용 시 자동으로 작동하며, 별도 설정 불필요

> 출처: https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-search-tool, https://medium.com/@joe.njenga/claude-code-just-cut-mcp-context-bloat-by-46-9-51k-tokens-down-to-8-5k-with-new-tool-search-ddf9e905f734

### 2.4 프롬프트 캐싱 전략 (Prompt Caching Strategies)

프롬프트 캐싱은 반복 사용되는 프롬프트 프리픽스(Prefix)를 캐시하여 처리 시간과 비용을 절감하는 기술이다. 캐시 읽기 토큰은 기본 입력 토큰 가격의 10분의 1이다.

#### 2.4.1 캐싱 가격 구조

| 모델 | 기본 입력 | 5분 캐시 쓰기 | 1시간 캐시 쓰기 | 캐시 히트 | 출력 |
|------|----------|-------------|---------------|---------|------|
| Claude Opus 4.6 | $5/MTok | $6.25/MTok (1.25배) | $10/MTok (2배) | $0.50/MTok (0.1배) | $25/MTok |
| Claude Sonnet 4.5 | $3/MTok | $3.75/MTok | $6/MTok | $0.30/MTok | $15/MTok |
| Claude Haiku 4.5 | $1/MTok | $1.25/MTok | $2/MTok | $0.10/MTok | $5/MTok |

**가격 승수(Multiplier):**
- 5분 캐시 쓰기 토큰: 기본 입력 가격의 **1.25배**
- 1시간 캐시 쓰기 토큰: 기본 입력 가격의 **2배**
- 캐시 읽기 토큰: 기본 입력 가격의 **0.1배** (즉, 90% 할인)

#### 2.4.2 캐싱 최적 사용 사례

| 사용 사례 | 효과 |
|----------|------|
| 대화형 에이전트: 긴 지시문 또는 업로드 문서가 포함된 확장 대화 | 비용/지연시간 대폭 감소 |
| 코딩 어시스턴트: 코드베이스 관련 섹션을 프롬프트에 유지 | 자동완성 및 코드 Q&A 성능 향상 |
| 대용량 문서 처리: 이미지 포함 전체 문서를 프롬프트에 포함 | 응답 지연시간 증가 없이 처리 |
| 에이전트 도구 사용: 다중 도구 호출과 반복적 코드 변경 | 각 API 호출의 효율성 향상 |

#### 2.4.3 실제 절약 사례

엔터프라이즈 문서 QA 시스템(일 1,000건 쿼리, 10개 문서(각 20,000 토큰), 문서당 100건 쿼리) 기준:
- 프롬프트 캐싱 적용 시 연간 약 **$20,000** 절약

#### 2.4.4 캐싱 제한사항

최소 캐시 가능 프롬프트 길이:
- Claude Opus 4.6, Opus 4.5: 4,096 토큰
- Claude Sonnet 4.5, Opus 4.1, Opus 4, Sonnet 4: 1,024 토큰
- Claude Haiku 4.5: 4,096 토큰
- Claude Haiku 3.5, Haiku 3: 2,048 토큰

기본 캐시 수명(TTL): 5분 (사용 시 자동 갱신, 추가 비용 없음). 1시간 캐시 옵션도 제공된다.

> 출처: https://platform.claude.com/docs/en/build-with-claude/prompt-caching, https://www.anthropic.com/news/prompt-caching

### 2.5 비용 추정 공식 (Cost Estimation Formulas)

#### 2.5.1 기본 비용 공식

```
총 비용 = (입력 토큰 수 x 입력 가격/MTok / 1,000,000)
         + (출력 토큰 수 x 출력 가격/MTok / 1,000,000)
```

예시 (Claude Sonnet 4.5 기준):
```
입력: 50,000 토큰, 출력: 10,000 토큰
비용 = (50,000 x $3 / 1,000,000) + (10,000 x $15 / 1,000,000)
     = $0.15 + $0.15
     = $0.30 / 요청
```

#### 2.5.2 캐싱 포함 비용 공식

```
총 비용 = (캐시 쓰기 토큰 x 캐시 쓰기 가격/MTok / 1,000,000)
         + (캐시 읽기 토큰 x 캐시 읽기 가격/MTok / 1,000,000)
         + (비캐시 입력 토큰 x 기본 입력 가격/MTok / 1,000,000)
         + (출력 토큰 x 출력 가격/MTok / 1,000,000)

총 입력 토큰 = cache_read_input_tokens + cache_creation_input_tokens + input_tokens
```

#### 2.5.3 에이전트 워크플로우 비용 추정 (Claude Opus 4.6 기준)

| 단계 | 일반적인 토큰 수 | 예상 비용 |
|------|----------------|----------|
| 초기 요청 처리 | 입력 500-1,000 | 약 $0.003/요청 |
| 메모리 및 컨텍스트 검색 | 검색 2,000-5,000 | 약 $0.015/작업 |
| 행동 계획 및 실행 | 계획 1,000-2,000 + 실행 피드백 500-1,000 | 약 $0.045/행동 |

**고객 지원 에이전트 사례:**
- 대화당 평균 약 3,700 토큰
- Claude Opus 4.6 ($5/MTok 입력, $25/MTok 출력) 기준
- 10,000건 지원 티켓 처리 비용: 약 **$37.00**

#### 2.5.4 추가 비용 절감 수단

| 방법 | 절감률 | 적용 조건 |
|------|--------|----------|
| 배치 API (Batch API) | 50% 할인 (입력+출력) | 비실시간 대량 처리 |
| 프롬프트 캐싱 | 최대 90% (입력 토큰) | 반복 프롬프트 |
| 모델 라우팅 | 50%+ | 다양한 복잡도의 태스크 혼합 |
| 노력 매개변수 (low) | 상당한 토큰 절약 | 단순 태스크, 서브에이전트 |
| 프로그래밍 방식 도구 호출 | 37% (복잡 태스크) | 다중 도구 호출 워크플로우 |

> 출처: https://platform.claude.com/docs/en/about-claude/pricing, https://www.silicondata.com/blog/llm-cost-per-token, https://medium.com/@alphaiterations/llm-cost-estimation-guide-from-token-usage-to-total-spend-fba348d62824

---

## 3. MCP 레지스트리 및 서버 디스커버리 (MCP Registry & Server Discovery)

### 3.1 MCP 레지스트리 개념

MCP 레지스트리(MCP Registry)는 공개적으로 사용 가능한 MCP 서버의 중앙화된 카탈로그(Centralized Catalog)이자 API이다. 2025년 9월 8일에 프리뷰로 출시되었으며, 공식 주소는 https://registry.modelcontextprotocol.io 이다.

**핵심 목표:**
- MCP 서버의 배포(Distribution)와 디스커버리(Discovery) 방법 표준화
- 서브 레지스트리(Sub-Registry)가 구축할 수 있는 일차적 진실의 원천(Primary Source of Truth) 역할
- 서버 도달 범위 확대, 클라이언트의 MCP 생태계 전반에서 서버 발견 지원

**아키텍처:**
- 오픈소스: MCP 레지스트리 자체와 상위 OpenAPI 명세가 오픈소스로 공개되어, 누구나 호환 가능한 서브 레지스트리 구축 가능
- API 기반: OpenAPI 명세를 기반으로 한 REST API 제공
- 네임스페이스 관리: 서버 게시(Publishing) 시 네임스페이스 소유권 검증, 스키마 적합성 검사

**API 엔드포인트 예시:**
- `GET /v0/servers` - 페이지네이션(Pagination)을 통한 서버 목록 조회
- 서버 메타데이터: 엔드포인트, 기능(Capabilities), 버전 관리 등의 필드 포함

> 출처: https://registry.modelcontextprotocol.io/, https://github.com/modelcontextprotocol/registry, http://blog.modelcontextprotocol.io/posts/2025-09-08-mcp-registry-preview/

### 3.2 서버 디스커버리 및 등록 과정

**서버 등록 과정:**
1. 서버 관리자가 MCP 서버를 설명하는 메타데이터(엔드포인트, 기능, 버전 관리, 관련 필드)를 작성
2. 레지스트리에 메타데이터를 게시(Publish)
3. 레지스트리가 최소한의 검증(네임스페이스 고유성, 스키마 적합성) 수행
4. 검증 통과 후 카탈로그에 등록

**디스커버리 메커니즘:**
- 중앙 레지스트리(Central Registry) → 서브 레지스트리(Sub-Registry) → 클라이언트(Client) 구조
- 조직이나 커뮤니티가 자체 기준에 따라 서브 레지스트리를 생성 가능 (예: 특정 MCP 클라이언트에 연관된 "MCP 마켓플레이스")
- 기존 커뮤니티와 기업이 구축한 레지스트리를 대체하지 않고 보완하는 관계

### 3.3 주요 MCP 마켓플레이스 및 플랫폼

#### 3.3.1 공식 MCP 레지스트리 (Official MCP Registry)

- 주소: https://registry.modelcontextprotocol.io/
- GitHub: https://github.com/modelcontextprotocol/registry
- 역할: 공개 MCP 서버의 일차적 진실의 원천
- 특징: 오픈소스 OpenAPI 명세, 서브 레지스트리 구축 지원

> 출처: http://blog.modelcontextprotocol.io/posts/2025-09-08-mcp-registry-preview/

#### 3.3.2 Smithery

Smithery는 MCP 서버를 발견하고 게시하는 중앙화된 허브이다.

**핵심 기능:**
- 레지스트리 웹사이트: https://smithery.ai/
- CLI 설치 도구: 커맨드라인에서 MCP 서버 설치 및 관리
- SDK: 개발자를 위한 소프트웨어 개발 키트
- 호스팅 서비스: MCP 서버 접근성을 위한 호스팅 지원
- 도구 통합을 위한 표준화된 인터페이스 및 설정 제공

> 출처: https://smithery.ai/, https://workos.com/blog/smithery-ai

#### 3.3.3 mcp.run

mcp.run은 보안적이고 이식 가능한(Portable) MCP 서버를 설치하고 실행하기 위한 호스팅 레지스트리이자 컨트롤 플레인(Control Plane)이다.

> 출처: https://mcp.run/

#### 3.3.4 Docker MCP Catalog

Docker가 제공하는 MCP 서버 전용 카탈로그로, Docker Hub와 통합된다.

**핵심 기능:**
- **검증된 서버**: mcp/ 네임스페이스 하의 모든 MCP 서버 이미지는 Docker가 빌드하고 디지털 서명. 소프트웨어 자재 명세서(SBOM) 포함
- **보안 격리**: 각 MCP 도구는 자체 컨테이너에서 실행 (1 CPU, 2GB 메모리 제한), 기본적으로 호스트 파일시스템 접근 불가
- **MCP Toolkit**: Docker Desktop에서 MCP 서버를 발견, 설정, 관리하는 그래픽 인터페이스
- **Dynamic MCP**: AI 에이전트가 대화 중에 수동 설정 없이 MCP 서버를 온디맨드로 발견, 추가, 조합
- **카탈로그 규모**: 200개 이상의 검증·버전 관리·큐레이션된 MCP 서버
- 레지스트리: https://github.com/docker/mcp-registry

> 출처: https://docs.docker.com/ai/mcp-catalog-and-toolkit/, https://www.docker.com/blog/announcing-docker-mcp-catalog-and-toolkit-beta/

### 3.4 프로덕션 MCP 서버 관리, 배포, 모니터링

#### 3.4.1 배포 아키텍처

**컨테이너 오케스트레이션 (Kubernetes):**
- 리소스 제한(Resource Limits), 상태 검사(Liveness/Readiness Probes) 설정
- 시크릿(Secrets)/ConfigMap에서 설정 로드
- 수평적 파드 오토스케일링(Horizontal Pod Autoscaling, HPA)

**AWS ECS 배포:**
- 다중 가용 영역(Availability Zone) 배포로 단일 장애 지점(Single Point of Failure) 제거
- 상태 검사와 로드 밸런서 통합으로 비정상 컨테이너 자동 교체
- CloudWatch 로그로 중앙화된 로깅 (설정 가능한 보존 기간)

**AWS 기반 배포 가이드:**
- https://aws.amazon.com/solutions/guidance/deploying-model-context-protocol-servers-on-aws/

#### 3.4.2 보안 및 접근 제어

- **인증(Authentication)**: OAuth 2.1 (2025년 6월 MCP 명세에 추가), SAML (엔터프라이즈 SSO), OIDC (현대적 ID 제공자), API 토큰 관리
- **권한 부여(Authorization)**: 공유 서비스 계정(Shared Service Account)과 사용자별 OAuth 플로우 모두 지원
- **입력 검증**: 엄격한 스키마 검증, 미지 필드 및 잘못된 요청 거부, 컨텍스트 기반 새니타이제이션(Sanitization), 시맨틱 검증으로 인젝션 및 매개변수 스머글링(Smuggling) 제한

#### 3.4.3 모니터링 및 관측성

필수 메트릭:
- **도구 호출 추적(Tool Call Tracking)**: 어떤 도구를 에이전트가 사용하는지 추적
- **성능 분석(Performance Analytics)**: 지연시간, 처리량
- **에러율(Error Rates)**: 실패한 도구 호출 비율
- **비용 할당(Cost Allocation)**: 팀별 비용 배분

#### 3.4.4 운영 회복성 (Operational Resilience)

- **멱등성(Idempotency)**: 도구 호출은 멱등적이어야 하며, 클라이언트 생성 요청 ID를 수용하고, 동일 입력에 대해 결정적(Deterministic) 결과 반환
- **페이지네이션**: 리스트 작업에 페이지네이션 토큰과 커서(Cursor) 사용, 응답 크기를 작고 예측 가능하게 유지
- **에러 처리**: 엄격한 스키마 검증, 미지 필드/잘못된 요청 거부, 입력 정규화

#### 3.4.5 MCP 게이트웨이 (MCP Gateway)

2026년에 등장한 엔터프라이즈 MCP 인프라의 핵심 컴포넌트로, MCP 클라이언트와 서버 사이에 위치하여 인증, 라우팅, 모니터링, 캐싱 등을 중앙에서 관리한다.

주요 MCP 게이트웨이:
- Docker MCP Toolkit 게이트웨이
- Cloudflare MCP 게이트웨이
- AWS 기반 커스텀 게이트웨이 구현

> 출처: https://thenewstack.io/15-best-practices-for-building-mcp-servers-in-production/, https://www.cdata.com/blog/mcp-server-best-practices-2026, https://www.mintmcp.com/blog/enterprise-ai-infrastructure-mcp, https://www.akto.io/blog/mcp-security-best-practices

### 3.5 MCP 레지스트리/마켓플레이스 비교

| 플랫폼 | 유형 | 핵심 특징 | 보안 | URL |
|--------|------|----------|------|-----|
| 공식 MCP Registry | 중앙 레지스트리 | 오픈소스 OpenAPI 명세, 서브 레지스트리 지원 | 네임스페이스 검증 | https://registry.modelcontextprotocol.io/ |
| Smithery | 마켓플레이스/허브 | CLI 설치, SDK, 호스팅 서비스 | 표준화 인터페이스 | https://smithery.ai/ |
| mcp.run | 호스팅 레지스트리 | 보안/이식 가능 서버, 컨트롤 플레인 | 샌드박스 실행 | https://mcp.run/ |
| Docker MCP Catalog | 컨테이너 레지스트리 | 200+ 검증 서버, SBOM, 디지털 서명 | 컨테이너 격리, 리소스 제한 | https://hub.docker.com/ (mcp/ 네임스페이스) |

---

## 참고문헌

### 관측성 플랫폼
- LangChain. (2026). LangSmith Plans and Pricing. https://www.langchain.com/pricing
- LangChain. (2026). LangSmith: AI Agent & LLM Observability Platform. https://www.langchain.com/langsmith/observability
- Arize AI. (2025). Arize Phoenix: AI Observability & Evaluation. https://github.com/Arize-ai/phoenix
- Arize AI. (2026). Phoenix Pricing. https://phoenix.arize.com/pricing/
- Langfuse. (2026). LLM Observability & Application Tracing. https://langfuse.com/docs/observability/overview
- Langfuse. (2026). Pricing. https://langfuse.com/pricing
- Langfuse. (2026). Self-host Langfuse. https://langfuse.com/self-hosting
- Braintrust. (2026). Pricing. https://www.braintrust.dev/pricing
- Helicone. (2026). Pricing. https://www.helicone.ai/pricing
- Helicone. (2025). Cost Tracking & Optimization. https://docs.helicone.ai/guides/cookbooks/cost-tracking
- lakeFS. (2026). LLM Observability Tools: 2026 Comparison. https://lakefs.io/blog/llm-observability-tools/
- Agenta. (2025). Top LLM Observability Platforms 2025. https://agenta.ai/blog/top-llm-observability-platforms

### 비용 최적화
- Anthropic. (2026). Pricing - Claude API Docs. https://platform.claude.com/docs/en/about-claude/pricing
- Anthropic. (2026). Prompt Caching. https://platform.claude.com/docs/en/build-with-claude/prompt-caching
- Anthropic. (2026). Effort. https://platform.claude.com/docs/en/build-with-claude/effort
- Anthropic. (2025). Extended Thinking. https://platform.claude.com/docs/en/build-with-claude/extended-thinking
- Anthropic. (2025). Token-saving updates on the Anthropic API. https://www.anthropic.com/news/token-saving-updates
- Anthropic. (2025). Advanced Tool Use. https://www.anthropic.com/engineering/advanced-tool-use
- Anthropic. (2026). Tool Search Tool. https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-search-tool
- Anthropic. (2025). Programmatic Tool Calling. https://platform.claude.com/docs/en/agents-and-tools/tool-use/programmatic-tool-calling
- AI Pricing Master. (2026). 10 AI Cost Optimization Strategies for 2026. https://www.aipricingmaster.com/blog/10-AI-Cost-Optimization-Strategies-for-2026
- Agent Framework Hub. (2026). AI Agent Production Costs 2026. https://www.agentframeworkhub.com/blog/ai-agent-production-costs-2026
- Silicon Data. (2026). Understanding LLM Cost Per Token. https://www.silicondata.com/blog/llm-cost-per-token
- Njenga, J. (2026). Claude Code Just Cut MCP Context Bloat by 46.9%. *Medium*. https://medium.com/@joe.njenga/claude-code-just-cut-mcp-context-bloat-by-46-9-51k-tokens-down-to-8-5k-with-new-tool-search-ddf9e905f734

### MCP 레지스트리
- Model Context Protocol. (2025). Introducing the MCP Registry. http://blog.modelcontextprotocol.io/posts/2025-09-08-mcp-registry-preview/
- Model Context Protocol. (2025). MCP Registry GitHub. https://github.com/modelcontextprotocol/registry
- Model Context Protocol. (2025). Official MCP Registry. https://registry.modelcontextprotocol.io/
- Smithery AI. (2026). Smithery. https://smithery.ai/
- WorkOS. (2026). Smithery AI: A Central Hub for MCP Servers. https://workos.com/blog/smithery-ai
- Docker. (2026). MCP Catalog. https://docs.docker.com/ai/mcp-catalog-and-toolkit/catalog/
- Docker. (2026). MCP Catalog and Toolkit. https://docs.docker.com/ai/mcp-catalog-and-toolkit/
- Docker. (2026). MCP Registry GitHub. https://github.com/docker/mcp-registry
- Nordic APIs. (2025). Getting Started With the Official MCP Registry API. https://nordicapis.com/getting-started-with-the-official-mcp-registry-api/
- The New Stack. (2026). 15 Best Practices for Building MCP Servers in Production. https://thenewstack.io/15-best-practices-for-building-mcp-servers-in-production/
- CData. (2026). MCP Server Best Practices for 2026. https://www.cdata.com/blog/mcp-server-best-practices-2026
- MintMCP. (2026). 7 Top MCP Gateways for Enterprise AI Infrastructure. https://www.mintmcp.com/blog/enterprise-ai-infrastructure-mcp
- Akto. (2026). Top MCP Security Best Practices for 2026. https://www.akto.io/blog/mcp-security-best-practices
- AWS. (2025). Guidance for Deploying Model Context Protocol Servers on AWS. https://aws.amazon.com/solutions/guidance/deploying-model-context-protocol-servers-on-aws/
