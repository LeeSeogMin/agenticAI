---
title: 주간 보고서 작성
description: GitHub 활동을 기반으로 주간 보고서를 생성한다
---
# 주간 보고서 작성 스킬

## 데이터 수집 절차
1. GitHub MCP의 list_pull_requests로 이번 주 merged PR을 조회한다
2. GitHub MCP의 list_pull_requests로 현재 open PR을 조회한다
3. GitHub MCP의 list_issues로 신규 및 미해결 이슈를 조회한다
4. 각 PR/이슈의 라벨을 확인하여 우선순위를 분류한다

## 보고서 구조
1. 기간 (이번 주 월요일~일요일)
2. 완료된 작업 (merged PR 기준)
3. 진행 중인 작업 (open PR 기준)
4. 신규/미해결 이슈 (라벨별 분류)
5. 다음 주 계획 (milestone 기반)

## 우선순위 분류 규칙
- critical/bug 라벨 → 긴급
- feature 라벨 → 일반
- docs/chore 라벨 → 낮음

## 작성 규칙
- 한국어로 작성한다
- 각 PR/이슈는 한 줄 요약으로 기술한다
- 날짜 형식: YYYY-MM-DD
- PR 번호를 반드시 포함한다 (예: #42)
- 담당자 이름을 괄호 안에 표기한다
