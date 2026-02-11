# 10장 리서치: 에이전트 보안 위협, 다층 가드레일, 평가 프레임워크

> 조사 기준일: 2026-02-11
> 대상 절: OWASP AI Agent Security Top 10, 다층 가드레일 아키텍처, 에이전트 평가 프레임워크

---

## 1. OWASP AI Agent Security Top 10 (2026)

### 개요
- **정식 명칭**: OWASP Top 10 for Agentic Applications 2026
- **발표일**: 2025년 12월 9일
- **개발 주체**: OWASP GenAI Security Project, 100명 이상의 보안 전문가·연구자·실무자 협력
- **공식 URL**: https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/
- **핵심 원칙**: 최소 에이전시(Least Agency) -- 에이전트에게 안전하고 제한된 작업 수행에 필요한 최소한의 자율성만 부여한다
- **기존 LLM Top 10과의 차이**: 기존 프레임워크는 "AI가 말하는 것(what AI says)"을 보호했다면, 에이전틱 Top 10은 "AI가 행하는 것(what AI does)"을 보호하는 데 초점을 둔다

### 완전한 위협 목록 (ASI01--ASI10)

| 순위 | 코드 | 위협명 | 설명 |
|------|------|--------|------|
| 1 | ASI01 | Agentic Goal Hijack (에이전트 목표 탈취) | 공격자가 지시, 도구 출력, 외부 콘텐츠를 조작하여 에이전트 목표를 전환시킨다. 프롬프트 인젝션의 에이전트 특화 확장이다. |
| 2 | ASI02 | Tool Misuse & Exploitation (도구 오용·악용) | 에이전트가 프롬프트 인젝션, 정렬 오류, 안전하지 않은 위임으로 인해 합법적 도구를 잘못 사용한다. |
| 3 | ASI03 | Identity & Privilege Abuse (신원·권한 남용) | 상속/캐시된 자격증명, 위임된 권한, 에이전트 간 신뢰를 악용한다. |
| 4 | ASI04 | Agentic Supply Chain Vulnerabilities (에이전틱 공급망 취약점) | 악의적이거나 변조된 도구, 디스크립터, 모델, 에이전트 페르소나가 실행을 침해한다. |
| 5 | ASI05 | Unexpected Code Execution (예기치 않은 코드 실행) | 에이전트가 공격자가 제어하는 코드를 생성하거나 실행한다. |
| 6 | ASI06 | Memory & Context Poisoning (메모리·컨텍스트 오염) | 에이전트 메모리, RAG 저장소, 컨텍스트 지식의 영구적 오염이다. |
| 7 | ASI07 | Insecure Inter-Agent Communication (안전하지 않은 에이전트 간 통신) | 위조, 조작, 가로챈 에이전트 통신이다. |
| 8 | ASI08 | Cascading Failures (연쇄 실패) | 단일 장애점이 멀티 에이전트 워크플로우 전체로 대규모 전파된다. |
| 9 | ASI09 | Human-Agent Trust Exploitation (인간-에이전트 신뢰 악용) | 설득력 있는 에이전트 설명이 인간 운영자를 오도하여 유해한 행동을 승인하게 한다. 에이전트에 대한 과도한 의존(over-reliance)이 안전하지 않은 승인이나 데이터 노출로 이어진다. |
| 10 | ASI10 | Rogue Agents (불량 에이전트) | 침해되거나 정렬 오류가 발생한 에이전트가 의도된 동작에서 이탈한다. 은닉, 자기 주도 행동, 의도와 다른 목표 추구 등의 패턴을 보인다. |

### 실제 사고 사례 (OWASP 트래커 기반)
- 에이전트 매개 데이터 유출 (agent-mediated data exfiltration)
- 원격 코드 실행 (RCE)
- 메모리 오염 (memory poisoning)
- 공급망 침해 (supply chain compromise)

### 교재 활용 시사점
- ASI01(목표 탈취)은 기존 프롬프트 인젝션의 에이전트 특화 진화형으로, 도구 출력을 통한 간접 인젝션(indirect injection) 경로가 핵심이다
- ASI02(도구 오용)와 ASI05(코드 실행)는 MCP 도구 호출의 보안 설계와 직결된다
- ASI06(메모리 오염)은 8-9장 RAG/GraphRAG의 보안 취약점으로 연결된다
- ASI08(연쇄 실패)은 7장 멀티 에이전트 시스템의 실패 모드와 관련된다

### 참고 자료
- [OWASP Top 10 for Agentic Applications 2026 (공식)](https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/)
- [OWASP GenAI Security Project 발표 블로그](https://genai.owasp.org/2025/12/09/owasp-top-10-for-agentic-applications-the-benchmark-for-agentic-security-in-the-age-of-autonomous-ai/)
- [OWASP GenAI Security Project - 릴리스 안내](https://genai.owasp.org/2025/12/09/owasp-genai-security-project-releases-top-10-risks-and-mitigations-for-agentic-ai-security/)
- [Palo Alto Networks - OWASP Agentic AI 분석](https://www.paloaltonetworks.com/blog/cloud-security/owasp-agentic-ai-security/)
- [Aikido - Full Guide to AI Agent Security Risks](https://www.aikido.dev/blog/owasp-top-10-agentic-applications)
- [BleepingComputer - Real-World Attacks Behind OWASP Agentic AI Top 10](https://www.bleepingcomputer.com/news/security/the-real-world-attacks-behind-owasp-agentic-ai-top-10/)
- [Tenable - OWASP Ranks Top Agentic AI App Risks](https://www.tenable.com/blog/cybersecurity-snapshot-owasp-agentic-ai-top-10-mitre-dangerous-software-weaknesses-12-12-2025)
- [NIST - Agentic AI: Emerging Threats, Mitigations, and Challenges (발표 자료)](https://csrc.nist.gov/csrc/media/presentations/2026/agentic-ai-emerging-threats,-mitigations,-and-cha/1.3-agentic_ai-sotiropoulos.pdf)

---

## 2. 다층 가드레일 아키텍처 (Multi-Layer Guardrail Architecture)

### 아키텍처 개요: 4개 방어 계층

```
사용자 입력 ──▶ [입력 가드레일] ──▶ [실행 샌드박싱] ──▶ [출력 가드레일] ──▶ 최종 응답
                    │                      │                      │
              PII 제거               gVisor/Firecracker       정책 규칙 검사
              탈옥 탐지               네트워크 이그레스          유해 콘텐츠 필터
              토큰 제한               허용목록                  PII 재검출
                    │                      │                      │
                    └──────── [ID/접근 제어: 에이전트별 최소 권한 IAM] ────────┘
```

### 2.1 입력 가드레일 (Input Guardrails)

#### PII 제거
- 정규식(regex) 패턴 매칭으로 사회보장번호, 신용카드번호, 이메일, 전화번호 등 식별
- ML 기반 NER(Named Entity Recognition) 모델로 문맥 의존적 PII 탐지
- 탐지 후 처리: 마스킹(예: "***-****-1234"), 토큰화, 또는 완전 제거
- 입력/출력 양방향에서 PII를 검사하는 것이 권장된다

#### 탈옥(Jailbreak) 탐지
- **Llama Prompt Guard 2**: 알려진 탈옥 공격 코퍼스 기반 분류기로 탈옥 시도를 탐지한다
- 규칙 기반(허용/거부 패턴, 프롬프트 정책)과 ML 기반(독성 탐지, 탈옥 탐지, PII 탐지) 분류기를 조합하는 것이 모범 사례이다
- LLM 게이트웨이에서 regex 및 토큰 매처 플러그인으로 대용량 프롬프트, 명백한 탈옥 시도, 노출된 PII를 차단한다
- **가드레일 우회 리스크**: 문자 주입 기법(character injection)과 감지 불가능한 적대적 ML 회피 공격(imperceptible adversarial evasion attacks)으로 프롬프트 인젝션 가드레일을 완전히 우회할 수 있다는 연구 결과가 있다 (arXiv:2504.11168)

### 2.2 실행 샌드박싱 (Execution Sandboxing)

#### 격리 기술 비교

| 기술 | 격리 수준 | 콜드 스타트 | 오버헤드 | 적합 시나리오 |
|------|-----------|------------|---------|-------------|
| 컨테이너 (Docker) | 프로세스 수준 (커널 공유) | ~수십 ms | 최소 | 신뢰할 수 있는 코드, 개발 환경 |
| gVisor (GKE Sandbox) | 애플리케이션 커널 (시스콜 인터셉트) | ~수백 ms | 10--20% | 쿠버네티스 내 비신뢰 코드 |
| Firecracker microVM | 하드웨어 수준 (KVM 기반) | ~125--150ms | <5 MiB/VM | 멀티 테넌트, 비신뢰 워크로드 (최적) |
| Kata Containers | microVM (컨테이너 UX) | ~수백 ms | 중간 | 컨테이너 호환성 필요 시 |
| WebAssembly (WASI) | 능력 기반 샌드박싱 | 거의 즉시 | 거의 없음 | 플러그인 능력 범위 제한 (생태계 미성숙) |

#### Firecracker microVM 상세
- AWS가 개발한 KVM 기반 경량 가상머신 모니터이다
- **콜드 스타트**: 125ms에 유저스페이스/애플리케이션 코드를 시작할 수 있으며, 호스트당 초당 최대 150개 microVM을 생성할 수 있다
- **메모리 효율**: 각 microVM이 5 MiB 미만의 메모리 오버헤드로 구동된다
- **보안**: 전용 커널로 워크로드를 격리하며, "jailer" 동반 프로그램이 가상화 장벽이 침해될 경우의 2차 방어선을 제공한다
- **실사례**: E2B가 Firecracker microVM으로 AI 에이전트 코드 실행 샌드박스를 제공하며, 150ms 콜드 스타트를 달성했다
- 하드웨어 수준 격리 + 밀리초 단위 시작 + 극히 낮은 오버헤드의 조합이 비신뢰 AI 코드 실행의 최적 표준(gold standard)으로 널리 인정된다

#### gVisor 상세
- 시스템 콜을 인터셉트하는 애플리케이션 커널로, 호스트 커널과 컨테이너 사이에 위치한다
- GKE Sandbox로 쿠버네티스 환경에서 사용 가능하다
- 10--20% 성능 오버헤드가 있지만 별도 VM 없이 강화된 격리를 제공한다

#### 네트워크 이그레스 허용목록 (Network Egress Allowlisting)
- 에이전트가 접근할 수 있는 외부 네트워크 대상을 FQDN(정규화 도메인명) 목록으로 제한한다
- **시행 모드**: enforced(정책 위반 연결 차단 및 로깅) 또는 dry-run(위반 허용하되 로깅만 수행) 모드를 지원한다
- Databricks는 서버리스 워크로드에 대해 이그레스 제어를 제공하며, 정책당 최대 100개 FQDN, 전체 2,500개 대상을 지원한다
- AWS Network Firewall Proxy(2025년 12월 프리뷰): NAT Gateway와 통합되어 TLS 인터셉션 및 중앙화된 모델을 지원하는 관리형 이그레스 보안 서비스이다
- Kubernetes 환경에서는 Calico 네트워크 정책, Istio 이그레스 게이트웨이 등으로 구현한다

### 2.3 출력 가드레일 (Output Guardrails)

- 에이전트 최종 응답에서 PII, PHI(보호 대상 건강 정보), 자격증명, 시크릿을 스캔하여 제거하거나 차단한다
- 정책 규칙 기반 검사: 유해 콘텐츠, 편향성, 규제 위반 등 조직 정책 위반 여부를 확인한다
- 에이전틱 RAG 환경에서는 입력/프로세스/출력 3계층 가드레일 시스템을 사용하여 PII, 독성, 환각을 실시간으로 검사한다

### 2.4 ID/접근 제어: 에이전트별 최소 권한 IAM

#### 핵심 문제
- 기존 IAM은 "누가(who)" 기준으로 권한을 관리하지만, AI 에이전트가 행동을 대행할 때는 에이전트의 신원에 대해 권한이 평가되어 요청자의 실제 권한과 불일치가 발생한다
- 조직의 에이전트는 개별 사용자보다 훨씬 넓은 권한으로 운영되는 경우가 대부분이다 -- 의도적 방치가 아니라 불확실성과 탐색 의지 때문이다

#### 최소 권한 접근법
1. **JIT(Just-In-Time) 접근**: ZSP(Zero Standing Privilege)와 JIT 접근을 에이전트에 적용하여 워크플로우를 방해하지 않으면서 최소 권한을 보장한다
2. **동적 정책 시행**: 초-분 단위 임시(ephemeral) 최소 권한 자격증명을 자동 프로비저닝하고, 상시 시크릿을 제거하며, 전체 감사 추적(사람/프롬프트/정책/행동/해체)을 캡처한다
3. **지속적 조정**: AI 에이전트의 최소 권한은 정적일 수 없으며, 관찰된 행동에 기반하여 지속적으로 조정해야 한다. 미사용 권한은 회수하고 상승된 접근은 임시적이고 목적에 한정되어야 한다

#### 산업 솔루션 (2025-2026)
- **CyberArk Secure AI Agents Solution**: AI 에이전트 신원 보호를 위한 업계 최초 권한 제어 솔루션으로, 에이전트가 필요한 접근만 필요할 때만 가지도록 보장한다
- **BeyondTrust**: AI 에이전트 권한 및 시크릿 보안 솔루션을 제공한다
- **Oasis Security**: 에이전틱 접근 관리(Agentic Access Management)를 출시하여 AI 에이전트용 최초 ID 솔루션을 제공한다

### 2.5 프레임워크 내장 가드레일

#### OpenAI Agents SDK Guardrails Primitive
- **공식 문서**: https://openai.github.io/openai-agents-python/guardrails/
- 가드레일이 SDK의 핵심 프리미티브(core primitive)로 포함되어 에이전트 입력 및 출력의 검증을 지원한다
- **입력 가드레일**: 사용자의 초기 입력에 대해 실행되며, 첫 번째 에이전트에서만 동작한다
- **출력 가드레일**: 에이전트의 최종 응답에 대해 실행되며, 마지막 에이전트에서만 동작한다
- **도구 가드레일(Tool Guardrails)**: 함수 도구를 감싸서 도구 호출 전후에 검증 또는 차단을 수행한다
- **구현 유형**: LLM 기반 에이전트(추론이 필요한 작업) 또는 규칙 기반/프로그래매틱 함수(특정 키워드 탐지 regex 등)로 구현 가능하다
- **실행 모드**:
  - 병렬 실행(기본값, `run_in_parallel=True`): 가드레일과 에이전트 실행이 동시에 진행되어 지연 시간을 최소화한다
  - 차단 실행(`run_in_parallel=False`): 가드레일이 완료된 후 에이전트가 시작되며, tripwire가 트리거되면 에이전트가 실행되지 않아 토큰 소비와 도구 실행을 방지한다

#### Google ADK Safety Settings
- **공식 문서**: https://google.github.io/adk-docs/safety/
- ADK는 에이전트 실행 파이프라인의 여러 구성요소에 보안 제어가 분산된 다층 보안 모델을 구현한다
- **가드레일 전략 4가지**:
  1. **인-도구 가드레일(In-Tool Guardrails)**: 도구를 방어적으로 설계하여, 개발자 설정 도구 컨텍스트로 정책을 시행한다 (예: 특정 테이블만 쿼리 허용)
  2. **내장 Gemini 안전 기능**: Gemini 모델 사용 시 유해 출력을 차단하는 콘텐츠 필터와 모델 동작을 안내하는 시스템 인스트럭션을 활용한다
  3. **콜백 및 플러그인**: 모델 및 도구 호출 전후에 에이전트 상태나 외부 정책 대비 매개변수를 검증한다
  4. **Gemini를 안전 계층으로 활용**: 저비용 고속 모델(Gemini Flash Lite 등)을 콜백으로 구성하여 입력과 출력을 스크리닝하는 추가 안전 계층을 구현한다
- **플러그인 기반 접근**: 특정 에이전트에 국한되지 않는 보안 정책은 플러그인으로 구현하는 것을 권장한다. 플러그인은 자체 완결적이고 모듈식이며, 러너(runner) 수준에서 전역 적용 가능하다
- **Google Cloud Model Armor**: 여러 위협 범주에 대한 사전 훈련된 분류기를 제공하는 엔터프라이즈급 안전 서비스로, 프로덕션 배포, 대량 처리, 일관되고 빠른 응답이 필요한 시나리오에 적합하다

### 2.6 확장: 4개 방어 평면 (Defense Planes)

포괄적 에이전트 보안은 4개 평면으로 구성된다:

1. **실행 평면 (Execution Plane)**: 에이전트는 gVisor/GKE Sandbox에서, 코드 실행 도구는 Firecracker microVM에서, 플러그인 능력 범위 제한은 WASI로, 네트워크 이그레스 허용목록은 네트워크 계층에서 시행한다
2. **데이터/메모리 평면 (Data/Memory Plane)**: 벡터 저장소는 프라이빗 서브넷에 배치하고, PII 편집은 수집·출력 양방향에서 수행하며, 메모리 TTL 및 주기적 삭제를 스케줄링한다
3. **관측성 평면 (Observability Plane)**: OpenTelemetry GenAI 스팬으로 프롬프트, 도구 호출, 안전 필터를 추적하고, SIEM에 집계하며, SOAR 자동화로 격리/권한 회수를 수행한다
4. **보증 평면 (Assurance Plane)**: Sigstore/Cosign 검증, SLSA 증명서와 SBOM을 CI/CD에 통합하고, MITRE 플레이북으로 레드팀 훈련을 주기적으로 수행하며, EU AI Act/NIST AI RMF 매핑으로 거버넌스 리뷰를 실시한다

### 참고 자료
- [Wiz - AI Guardrails: Safety Controls for Responsible AI Use](https://www.wiz.io/academy/ai-security/ai-guardrails)
- [Datadog - LLM Guardrails Best Practices](https://www.datadoghq.com/blog/llm-guardrails-best-practices/)
- [TechWink - Three-Layer Guardrail for Agentic RAG](https://techwink.net/blog/three-layer-guardrail-for-agentic-rag-best-practices-for-2026/)
- [CodeAnt - How to Sandbox LLMs & AI Shell Tools (Docker, gVisor, Firecracker)](https://www.codeant.ai/blogs/agentic-rag-shell-sandboxing)
- [Northflank - How to Sandbox AI Agents in 2026](https://northflank.com/blog/how-to-sandbox-ai-agents)
- [Northflank - Best Code Execution Sandbox for AI Agents 2026](https://northflank.com/blog/best-code-execution-sandbox-for-ai-agents)
- [Firecracker 공식 사이트](https://firecracker-microvm.github.io/)
- [OpenAI Agents SDK Guardrails 공식 문서](https://openai.github.io/openai-agents-python/guardrails/)
- [Google ADK Safety 공식 문서](https://google.github.io/adk-docs/safety/)
- [HackerNews - AI Agents Are Becoming Authorization Bypass Paths](https://thehackernews.com/2026/01/ai-agents-are-becoming-privilege.html)
- [ISACA - The Looming Authorization Crisis: Why Traditional IAM Fails Agentic AI](https://www.isaca.org/resources/news-and-trends/industry-news/2025/the-looming-authorization-crisis-why-traditional-iam-fails-agentic-ai)
- [CyberArk - AI Agent Identity Security Solution](https://www.cyberark.com/press/cyberark-introduces-first-identity-security-solution-purpose-built-to-protect-ai-agents-with-privilege-controls/)
- [arXiv:2504.11168 - Bypassing Prompt Injection and Jailbreak Detection in LLM Guardrails](https://arxiv.org/html/2504.11168v1)
- [Skywork - Agentic AI Safety Best Practices 2025](https://skywork.ai/blog/agentic-ai-safety-best-practices-2025-enterprise/)
- [Superagent Framework for Guardrails (Help Net Security)](https://www.helpnetsecurity.com/2025/12/29/superagent-framework-guardrails-agentic-ai/)

---

## 3. 에이전트 평가 프레임워크 (Agent Evaluation Frameworks)

### 3.1 GAIA: 범용 AI 어시스턴트 벤치마크

- **정식 명칭**: GAIA -- a benchmark for General AI Assistants
- **논문**: Mialon et al. (2023). "GAIA: a benchmark for General AI Assistants." *arXiv*. https://arxiv.org/abs/2311.12983
- **리더보드**: https://hal.cs.princeton.edu/gaia
- **개발**: Meta AI 외 공동 연구

#### 핵심 특성
- 인간에게 개념적으로 단순하지만 AI에게 어려운 실세계 질문 450개로 구성된다
- 각 질문은 명확한 정답이 있어 자동 평가가 가능하다
- 추론, 멀티모달 처리, 웹 브라우징, 도구 사용 능력을 종합 측정한다
- 에이전트가 자율적으로 여러 단계를 계획(plan), 결정(decide), 실행(act)하는 **멀티스텝 계획 능력**을 핵심적으로 평가한다

#### 3단계 난이도
| 레벨 | 설명 | 단계 수 |
|------|------|---------|
| Level 1 | 숙련된 LLM이 해결 가능, 최소 도구 사용 | 5단계 미만 |
| Level 2 | 복잡한 추론과 다수 도구의 적절한 사용 필요 | 5--10단계 |
| Level 3 | 장기 계획과 다양한 도구의 정교한 통합 필요 (모델 능력의 큰 도약 지표) | 10단계 이상 |

#### 성능 진화
- **초기(2023)**: 인간 92% vs GPT-4(플러그인) 15%
- **2025**: H2O.ai h2oGPTe Agent가 테스트셋에서 75% 정확도 달성 -- GAIA에서 최초로 C 등급 도달
- **Level 3**: Action Agent가 61% 달성으로 1위

### 3.2 WebArena: 웹 상호작용 평가

- **정식 명칭**: WebArena -- A Realistic Web Environment for Building Autonomous Agents
- **논문**: Zhou et al. (2023). "WebArena: A Realistic Web Environment for Building Autonomous Agents." *arXiv*. https://arxiv.org/abs/2307.13854
- **사이트**: https://webarena.dev/
- **GitHub (Verified)**: https://github.com/ServiceNow/webarena-verified

#### 핵심 특성
- 자체 호스팅 가능한 완전 재현 가능 웹 환경으로, 주요 웹사이트 범주의 현실적·인터랙티브 시뮬레이션을 제공한다
- **4개 도메인**: 전자상거래, 소셜 포럼, 협업 소프트웨어 개발, 콘텐츠 관리
- **812개 장기 호라이즌 과제**: 241개 템플릿 기반, 템플릿당 평균 3.3개 변형
- 계획 수립, 내비게이션, 메모리 관리, 도구 조정 등의 능력을 측정한다
- 평가 기준: 기능적·종단간(end-to-end) 과제 정확성

#### 성능 진화
- **2023**: 14% 성공률
- **2025**: 약 60% 성공률로 도약, 단일 에이전트 최고 기록 61.7% 달성 (AWA 1.5)

#### 최근 확장 (2025-2026)
- **WebArena-Verified**: 큐레이션된 버전 관리 과제 데이터셋과 결정론적 평가기를 제공하는 검증 릴리스이다. 2026년 1월 PyPI에서 `pip install webarena-verified`로 설치 가능하다.
- **WebChoreArena**: 532개의 정교하게 큐레이션된 과제로 WebArena의 범위를 일반 브라우징 너머 노동 집약적이고 반복적인 과제로 확장한다.

### 3.3 Terminal-Bench: CLI 워크플로우 평가

- **정식 명칭**: Terminal-Bench -- Benchmarking Agents on Hard, Realistic Tasks in Command Line Interfaces
- **논문**: arXiv:2601.11868 (2026년 1월)
- **사이트**: https://www.tbench.ai/
- **GitHub**: https://github.com/laude-institute/terminal-bench
- **공동 개발**: Stanford University, Laude Institute

#### 핵심 특성
- 89개 과제(v2.0)로 구성된 정교하게 큐레이션된 어려운 벤치마크이다
- 실제 워크플로우에서 영감을 받은 컴퓨터 터미널 환경의 과제를 포함한다
- 각 과제는 고유한 환경, 사람이 작성한 솔루션, 검증을 위한 포괄적 테스트를 갖추고 있다
- Core set (v0.1.1): 80개 인간 검증, Docker화된 과제

#### 과제 범위
- 소프트웨어 개발 및 코딩 도전
- 시스템 관리
- 빌드/테스트 및 의존성 관리
- 데이터 및 ML 워크플로우
- 시스템 및 네트워킹
- 보안
- 핵심 CLI 워크플로우
- 예시: Fortran 빌드 프로세스 현대화, git 웹 서버 구성, RL 에이전트/텍스트 분류기 훈련, Conda 의존성 충돌 해결, 레포지토리 시크릿 정리

#### 성능 (2025-2026)
| 에이전트 | 모델 | 점수 |
|---------|------|------|
| Droid | Opus | 58.8% |
| Droid | GPT-5 | 52.5% |
| Droid | Sonnet | 50.5% |
| Claude Code | Opus | 43.2% |
| Codex CLI | - | 42.8% |

- 프론티어 모델과 에이전트도 65% 미만의 점수를 기록한다
- GitHub 스타 1,000개 이상, 전세계 약 100명 개발자 기여

### 3.4 CLEAR: 비용/지연/효능/보증/신뢰성 프레임워크

- **정식 명칭**: Beyond Accuracy: A Multi-Dimensional Framework for Evaluating Enterprise Agentic AI Systems
- **논문**: arXiv:2511.14136 (2025년 11월)
- **URL**: https://arxiv.org/abs/2511.14136

#### 5개 차원

| 차원 | 약어 | 측정 대상 | 주요 메트릭 |
|------|------|----------|-----------|
| Cost (비용) | C | API 토큰 소비, 추론 비용, 인프라 오버헤드 | CNA (Cost-Normalized Accuracy): 고비용 고정확도 vs 비용 효율적 대안을 공정 비교 |
| Latency (지연) | L | 계획/실행/반영 단계별 응답 시간 | 종단간 과제 완료 시간, 도메인별 SLA 임계값 기반 SLA 준수율 |
| Efficacy (효능) | E | 과제 완료 품질 | 전통적 정확도 + 도메인 특화 측정 |
| Assurance (보증) | A | 안전·보안·정책 준수 | 프롬프트 인젝션 저항성, 데이터 유출 방지, 환각률, 우아한 실패 처리 |
| Reliability (신뢰성) | R | 다회 실행 일관성 | pass@k: 단일 실행 대비 k회 실행의 일관성 평가 |

#### 핵심 발견
1. **비용 격차**: 정확도만 최적화하면 유사한 성능 대비 4.4--10.8배 비싼 에이전트가 만들어진다
2. **신뢰성 하락**: 에이전트 성능이 단일 실행 60%에서 8회 실행 일관성 25%로 급락한다
3. **50배 비용 변이**: 유사 정밀도에서도 비용이 50배 차이날 수 있다
4. **전문가 검증**: 전문가 평가(N=15)에서 CLEAR가 프로덕션 성공을 더 정확히 예측한다 (상관계수 rho=0.83 vs 정확도만 사용 시 rho=0.41)
5. **평가 규모**: 6개 선도 에이전트를 300개 엔터프라이즈 과제에서 평가하였다

#### 기존 벤치마크의 3가지 근본 한계 (CLEAR가 해결)
1. 비용 제어 평가의 부재로 유사 정밀도에서 50배 비용 변이가 발생한다
2. 불충분한 신뢰성 평가로 단일 실행 vs 다회 실행 일관성의 성능 격차가 드러나지 않는다
3. 보안, 지연, 정책 준수에 대한 다차원 메트릭이 누락되어 있다

### 3.5 LLM-as-Judge

- **핵심 서베이 논문**:
  - Gu et al. (2024). "A Survey on LLM-as-a-Judge." *arXiv*. https://arxiv.org/abs/2411.15594
  - Haitao et al. (2024). "LLMs-as-Judges: A Comprehensive Survey on LLM-based Evaluation Methods." *arXiv*. https://arxiv.org/abs/2412.05579

#### 개념
- LLM을 평가자로 활용하여 복잡한 과제의 AI 출력을 평가하는 패러다임이다
- 전통적 전문가 평가의 대안으로, 다양한 데이터 유형을 처리하고 확장 가능하며 비용 효율적이고 일관된 평가를 제공한다
- 인간 판단과 약 90% 일치율을 달성한다

#### 조직 채택 현황
- `contents.md`에 명시된 "조직 59.8% 사용" 통계의 정확한 출처는 공개 검색에서 확인되지 않았다. 가장 근접한 데이터는 Monte Carlo Data의 2025년 3월 내부 조사에서 "데이터+AI 팀의 40%가 AI 에이전트를 프로덕션 단계에 두고 있으며 LLM-as-Judge를 모니터링에 활용"이라는 결과이다
- McKinsey 조사(2025): 78%의 조직이 최소 1개 사업 기능에 AI를 사용하며, 67%가 LLM을 활용한 생성형 AI를 운영에 도입했다
- **주의**: 59.8% 수치는 특정 산업 보고서(예: 컨설팅 사, 리서치 기관)에서 출처한 것으로 추정되나, 공개 검색으로 원본을 확인하지 못했다. 집필 시 출처 재확인이 필요하다

#### 주요 과제
- 편향 완화(bias mitigation)
- 프롬프트 엔지니어링 표준화
- 평가 방법론의 정형화
- 체계적 리뷰와 공식 정의의 부재가 연구자·실무자의 활용을 저해한다

### 3.6 기타 주목할 만한 에이전트 평가 벤치마크

#### tau-bench (tau-Bench)
- **개발**: Sierra AI
- **논문**: arXiv:2406.12045
- **사이트**: https://sierra.ai/blog/benchmarking-ai-agents
- 동적 대화에서 사용자(LM 시뮬레이션)와 도메인 특화 API 도구 및 정책 가이드라인이 제공된 언어 에이전트 간 상호작용을 평가한다
- 기존 벤치마크가 다루지 않는 인간 사용자와의 상호작용 및 도메인 특화 규칙 준수 능력을 테스트한다
- **메트릭**: pass^k -- k회 시행에서의 에이전트 행동 신뢰성을 측정한다
- **성능**: GPT-4o도 과제의 50% 미만 성공, pass^8 기준 소매 도메인에서 25% 미만
- **tau2-bench (2025)**: 코드 수정과 통신(telecom) 도메인을 추가한 확장판이다

#### SWE-Bench 패밀리
- **SWE-Bench**: 12개 인기 Python 저장소의 GitHub 이슈에서 수집한 2,294개 실세계 소프트웨어 엔지니어링 문제로 구성된다
- **SWE-Bench Lite**: 300개 인스턴스로 구성된 큐레이션 서브셋으로 버그 수정 능력을 평가한다
- **SWE-Bench Verified**: OpenAI와 전문 개발자가 공동으로 인간 검증한 500개 샘플이다
- **SWE-PolyBench (Amazon)**: 다중 프로그래밍 언어에 걸친 폴리글랏 코드베이스 처리 능력을 평가한다

#### AgentBench
- LLM-as-Agent의 멀티턴 개방형 설정에서의 추론 및 의사결정 능력을 평가한다
- 8개 환경: 운영체제, 데이터베이스, 지식 그래프, 디지털 카드 게임, 측면 사고 퍼즐, 가사 관리, 웹 쇼핑, 웹 브라우징

#### BFCL (Berkeley Function Calling Leaderboard)
- 직렬, 병렬, 멀티턴 상호작용 등 다양한 시나리오에서 LLM의 함수 호출 능력을 평가하는 포괄적 벤치마크이다
- 추론, 메모리, 웹 검색, 형식 민감도 등 에이전틱 능력을 측정한다

#### DPAI Arena (JetBrains, 2025년 10월)
- 멀티 워크플로우, 멀티 언어 개발자 에이전트를 평가하는 광범위 플랫폼이다
- 패치 생성, 테스트 작성, PR 리뷰, 정적 분석, 낯선 저장소 탐색 등을 평가한다

#### Context-Bench (Letta, 2025년 10월)
- 장기 실행 컨텍스트의 유지, 재사용, 추론 능력을 평가한다
- 파일 연산 체이닝과 장기 워크플로우에서의 일관된 의사결정을 테스트한다

### 참고 자료
- [GAIA 논문 (arXiv:2311.12983)](https://arxiv.org/abs/2311.12983)
- [GAIA 리더보드 (Princeton HAL)](https://hal.cs.princeton.edu/gaia)
- [H2O.ai - GAIA 벤치마크 1위 달성](https://h2o.ai/blog/2025/h2o-ai-tops-the-general-ai-assistant-test/)
- [WebArena 논문 (arXiv:2307.13854)](https://arxiv.org/abs/2307.13854)
- [WebArena 공식 사이트](https://webarena.dev/)
- [WebArena-Verified (GitHub)](https://github.com/ServiceNow/webarena-verified)
- [AWA 1.5 WebArena 성과 (Jace AI)](https://jace.ai/blog/awa-1-5-achieves-breakthrough-performance-on-web-arena-benchmark)
- [Terminal-Bench 논문 (arXiv:2601.11868)](https://arxiv.org/abs/2601.11868)
- [Terminal-Bench 공식 사이트](https://www.tbench.ai/)
- [Terminal-Bench GitHub](https://github.com/laude-institute/terminal-bench)
- [Terminal-Bench 2.0 (Snorkel AI)](https://snorkel.ai/blog/terminal-bench-2-0-raising-the-bar-for-ai-agent-evaluation/)
- [CLEAR 프레임워크 논문 (arXiv:2511.14136)](https://arxiv.org/abs/2511.14136)
- [LLM-as-a-Judge 서베이 (arXiv:2411.15594)](https://arxiv.org/abs/2411.15594)
- [LLMs-as-Judges 종합 서베이 (arXiv:2412.05579)](https://arxiv.org/abs/2412.05579)
- [tau-bench (Sierra AI)](https://sierra.ai/blog/benchmarking-ai-agents)
- [tau-bench 논문 (arXiv:2406.12045)](https://arxiv.org/abs/2406.12045)
- [o-mega - 2025-2026 AI Agent Benchmarks Guide](https://o-mega.ai/articles/the-best-ai-agent-evals-and-benchmarks-full-2025-guide)
- [8 Benchmarks Shaping the Next Generation of AI Agents](https://ainativedev.io/news/8-benchmarks-shaping-the-next-generation-of-ai-agents)
- [AI Agent Benchmark Compendium (GitHub)](https://github.com/philschmid/ai-agent-benchmark-compendium)
- [Evidently AI - 10 AI Agent Benchmarks](https://www.evidentlyai.com/blog/ai-agent-benchmarks)
- [Monte Carlo Data - LLM-as-Judge Best Practices](https://www.montecarlodata.com/blog-llm-as-judge/)

---

## 4. 미검증 사항 및 집필 시 주의사항

1. **LLM-as-Judge 59.8% 통계**: `contents.md`에 명시된 "조직 59.8% 사용" 수치의 정확한 출처를 공개 웹 검색으로 확인하지 못했다. 집필 시 원 출처를 재확인하거나, 확인 불가 시 대체 통계(McKinsey 78% AI 도입률 등)를 사용하는 것을 권장한다.
2. **OWASP Top 10 버전**: 정식 명칭은 "OWASP Top 10 for Agentic Applications for 2026"이며 발표일은 2025년 12월 9일이다. "AI Agent Security Top 10"은 비공식 약칭이다.
3. **가드레일 우회 리스크**: 입력 가드레일의 우회 가능성에 대한 최신 연구(arXiv:2504.11168)를 교재에 반영하여, 가드레일이 완벽하지 않다는 점을 독자에게 전달해야 한다.
4. **격리 기술 선택**: Firecracker가 최적 표준이지만 Linux/KVM 전용이라는 제약이 있다. macOS 개발 환경에서의 대안(Docker + 제한된 seccomp 프로파일 등)도 언급이 필요하다.
