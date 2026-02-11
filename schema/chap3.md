# 제3장: MCP 개념과 도구/리소스 설계 — 집필 계획서

## 장 개요

이 장은 MCP(Model Context Protocol)를 "특정 제품 기능"이 아니라, 에이전트가 외부 세계와 상호작용하기 위한 **표준화된 인터페이스(도구/리소스)** 관점에서 소개한다. v2.0에서 MCP 2025-11-25 스펙, 생태계, AAIF 거버넌스 섹션을 추가하여 프로토콜의 진화와 현재 위상을 반영한다.

### 학습 목표

1) MCP에서 tool/resource가 각각 무엇을 의미하는지 설명한다.
2) MCP 스펙의 진화 과정과 2025-11-25 핵심 변경 사항을 설명할 수 있다.
3) 좋은 tool 인터페이스의 조건(입출력/에러/제약/멱등성/보안)을 체크리스트로 정리한다.
4) 구현에 앞서 "도구 명세(JSON)"를 작성하고, 이를 기반으로 최소 스켈레톤 서버를 구성한다.

### 장 구성

- 3.1 왜 MCP가 필요한가
- 3.2 MCP의 진화: 2025 스펙과 생태계 [신규]
- 3.3 MCP 서버의 기본 구조
- 3.4 도구(Tool) 설계 원칙
- 3.5 리소스(Resource) 설계
- 3.6 실습: 최소 MCP 서버 구현

---

## 3.1 왜 MCP가 필요한가

### 집필 방향

"도구를 붙인다"는 말을 기능 나열로 풀지 않고, 운영 관점(일관된 I/O, 실패 처리, 감사 가능성)으로 풀어 설명한다. 제1장의 핵심(검증 가능한 산출물)과 연결해, MCP가 단순 편의 기능이 아니라 '검증 가능한 도구 호출'을 가능케 하는 구조라는 점을 강조한다.

---

## 3.2 MCP의 진화: 2025 스펙과 생태계 [신규]

### 집필 방향

MCP 스펙의 진화(2024-11-05 → 2025-11-25)를 표로 정리하고, 2025-11-25 핵심 변경(Tasks, Structured Outputs, Elicitation, Server Discovery)을 서술한다. 생태계(10,000+ 서버, 97M SDK 다운로드, 플랫폼 지원)와 AAIF 거버넌스를 포함한다.

### 표(필수)

- **표 3.1** MCP 스펙 버전별 주요 변경
- **표 3.2** 주요 플랫폼 MCP 지원 현황

### 강화 항목(반드시 포함)

- Tasks primitive (비동기 "call-now, fetch-later" 패턴, 5가지 상태)
- Structured Tool Outputs (outputSchema + structuredContent)
- Elicitation (폼 모드 + URL 모드)
- AAIF (Anthropic·Block·OpenAI 공동 설립, Linux Foundation 산하)

---

## 3.3 MCP 서버의 기본 구조

### 집필 방향

STDIO 통신 방식, JSON-RPC 2.0 프로토콜, 서버 생명주기, 최소 서버 구현을 설명한다.

---

## 3.4 도구(Tool) 설계 원칙

### 집필 방향

좋은 tool은 "기능"보다 "계약(contract)"이 먼저라는 흐름으로, 입출력·에러·제약을 서술한다.

---

## 3.5 리소스(Resource) 설계

### 집필 방향

리소스를 "파일처럼 읽는 컨텍스트"로 설명하고, tool과 비교해 언제 resource가 유리한지를 설명한다.

---

## 3.6 실습: 최소 MCP 서버 구현

### 실습 산출물(필수)

- `practice/chapter3/data/output/ch03_tool_spec.json`
- `practice/chapter3/data/output/ch03_demo_request.json`
- `practice/chapter3/data/output/ch03_demo_response.json`

### 실습 코드

- `practice/chapter3/code/3-5-minimal-mcp-server.py`

---

## 품질 기준(체크)

- [x] 학습목표 4개
- [x] 표 2개 (표 3.1 스펙 비교, 표 3.2 플랫폼 지원)
- [x] [신규] MCP 2025 스펙 핵심 변경
- [x] [신규] MCP 생태계 현황
- [x] [신규] MCP 거버넌스 (AAIF)
- [x] 실습 코드 참조
- [x] 참고문헌 (검증 가능한 실재 자료)

---

**마지막 업데이트**: 2026-02-11
**버전**: v2.0
