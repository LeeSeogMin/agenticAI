# 7장 리서치 결과: 멀티에이전트 개요와 선택 전략

## 조사일: 2026-01-02

## 1. 멀티에이전트 시스템 개요

### 정의
여러 AI 에이전트가 협력하여 복잡한 작업을 수행하는 시스템. 각 에이전트는 전문 역할을 맡아 분업하고 협력한다.

### 단일 에이전트 vs 멀티에이전트

**단일 에이전트 적합:**
- 단순하고 범위가 명확한 작업
- 문서 요약, 이메일 응답, 특정 정보 검색
- 빠른 응답 필요 시
- 비용 최소화 필요 시

**멀티에이전트 적합:**
- 복잡한 다단계 작업
- 환각(hallucination) 감소 필요 시 (상호 검증)
- 긴 컨텍스트 처리
- 병렬 처리로 속도 향상 필요 시

### 트레이드오프
- 비용 증가: 에이전트 추가 시 API 호출 증가
- 지연 시간: 에이전트 간 대기 시 응답 시간 증가
- 디버깅 복잡성: 문제 발생 지점 파악 어려움
- 무한 루프 위험: 에이전트 간 반복 호출

## 2. CrewAI

### 개요
- Python 기반 멀티에이전트 프레임워크
- 역할 기반 에이전트 설계
- LangChain과 독립적으로 개발됨
- Python 3.10~3.13 지원

### 핵심 구성 요소
1. **Agents**: 특정 역할과 목표를 가진 AI 개체
2. **Tasks**: 에이전트가 수행할 작업 정의
3. **Crews**: 함께 작업하는 에이전트 팀
4. **Tools**: 에이전트가 사용할 외부 도구
5. **Processes**: 협업 워크플로우 정의

### 특징
- YAML 기반 에이전트 설정 (권장)
- 구조화된 역할 기반 접근
- 빠른 프로토타이핑에 적합

## 3. AutoGen (Microsoft)

### 개요
- Microsoft Research에서 개발
- 대화 기반 멀티에이전트 프레임워크
- AutoGen v0.4 (2025년 1월): 이벤트 드리븐 아키텍처로 재설계

### v0.4 주요 기능
- 비동기 메시징
- 모듈형 확장 설계
- Python, .NET 지원
- GroupChat, 이벤트 드리븐 런타임

### Microsoft Agent Framework (2025)
- Semantic Kernel + AutoGen 통합
- 엔터프라이즈급 기능
- 워크플로우 명시적 제어
- Human-in-the-loop 지원

### 개발 도구
- AutoGen Studio: 노코드 GUI
- AutoGen Bench: 벤치마킹 도구

## 4. 프레임워크 선택 기준

| 기준 | CrewAI | AutoGen | LangGraph |
|-----|--------|---------|-----------|
| 설계 철학 | 역할 기반 팀 | 대화 기반 협업 | 그래프 워크플로우 |
| 학습 곡선 | 낮음 | 중간 | 높음 |
| 제어 수준 | 추상화 높음 | 중간 | 세밀한 제어 |
| Human-in-loop | 기본 지원 | 강력 지원 | 커스텀 구현 |

## 5. 2025년 트렌드

- 프론티어 LLM(o3, Gemini 2.5 Pro) 발전으로 단일 에이전트 역량 향상
- 그러나 복잡한 작업에서 멀티에이전트 여전히 우위
- 간단한 작업부터 시작, 점진적 복잡성 증가 권장

## 참고 자료

- CrewAI Documentation: https://docs.crewai.com/
- CrewAI GitHub: https://github.com/crewAIInc/crewAI
- Microsoft AutoGen: https://github.com/microsoft/autogen
- AutoGen Documentation: https://microsoft.github.io/autogen/
- Microsoft Agent Framework: https://learn.microsoft.com/en-us/agent-framework/overview/
- SuperAnnotate Multi-agent LLMs: https://www.superannotate.com/blog/multi-agent-llms
