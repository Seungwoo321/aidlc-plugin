<!--
  Copyright (c) revfactory (https://github.com/revfactory/harness)
  Licensed under the Apache License, Version 2.0
  https://www.apache.org/licenses/LICENSE-2.0

  Modified by Seungwoo Lee for AIDLC Plugin.
  Changes: AIDLC 방법론 맥락에 맞게 수정, 요약, AIDLC 프로젝트 예시 추가
-->

# 오케스트레이터 스킬 템플릿

> 출처: revfactory/harness (Apache-2.0), AIDLC aidlc-run 연동용

오케스트레이터는 팀 전체를 조율하는 상위 스킬이다. aidlc-run이 이 템플릿을 참조하여 도메인 에이전트를 조율한다.

---

## 템플릿 A: 에이전트 팀 모드 (기본)

```markdown
---
name: {domain}-orchestrator
description: "{도메인} 에이전트 팀을 조율하는 오케스트레이터."
---

# {Domain} Orchestrator

## 실행 모드: 에이전트 팀

## 에이전트 구성

| 팀원 | 에이전트 타입 | 역할 | 출력 |
|------|-------------|------|------|
| {teammate-1} | {커스텀} | {역할} | {output-file} |
| {teammate-2} | {커스텀} | {역할} | {output-file} |

## 워크플로우

### Phase 1: 준비
1. 사용자 입력 분석
2. `_workspace/` 생성
3. 입력 데이터를 `_workspace/00_input/`에 저장

### Phase 2: 팀 구성
1. TeamCreate로 팀 생성
2. TaskCreate로 작업 등록 (팀원당 5~6개 적정)

### Phase 3: 주요 작업
팀원들이 자체 조율. 리더는 모니터링 + 필요 시 개입.

**팀원 간 통신 규칙:**
- {teammate-1}은 {teammate-2}에게 {정보}를 SendMessage로 전달
- 작업 완료 시 결과를 파일로 저장 + 리더에게 알림

### Phase 4: 통합/검증
1. 모든 팀원 작업 완료 대기 (TaskGet)
2. 산출물 Read로 수집
3. 최종 산출물 생성

### Phase 5: 정리
1. 팀원 종료 요청
2. TeamDelete
3. `_workspace/` 보존 (감사 추적용)
4. 결과 요약 보고

## 데이터 흐름

```
[리더] → TeamCreate → [teammate-1] ←SendMessage→ [teammate-2]
                          ↓                           ↓
                    artifact-1.md              artifact-2.md
                          └───────── Read ────────────┘
                                     ↓
                              [리더: 통합] → 최종 산출물
```

## 에러 핸들링

| 상황 | 전략 |
|------|------|
| 팀원 1명 실패 | 상태 확인 → 재시작 또는 대체 팀원 |
| 팀원 과반 실패 | 사용자에게 알리고 진행 여부 확인 |
| 타임아웃 | 부분 결과 사용, 미완료 팀원 종료 |
| 데이터 충돌 | 출처 명시 후 병기 |
```

---

## 템플릿 B: 서브 에이전트 모드 (경량)

```markdown
---
name: {domain}-orchestrator
description: "{도메인} 에이전트를 조율하는 오케스트레이터."
---

# {Domain} Orchestrator

## 실행 모드: 서브 에이전트

## 워크플로우

### Phase 1: 준비
1. 사용자 입력 분석
2. `_workspace/` 생성

### Phase 2: 주요 작업
단일 메시지에서 N개 Agent 도구를 동시 호출:

| 에이전트 | subagent_type | 출력 | model | run_in_background |
|---------|--------------|------|-------|-------------------|
| {agent-1} | {타입} | `_workspace/{artifact}.md` | opus | true |
| {agent-2} | {타입} | `_workspace/{artifact}.md` | opus | true |

### Phase 3: 통합
1. 산출물 Read로 수집
2. 최종 산출물 생성

### Phase 4: 정리
1. `_workspace/` 보존
2. 결과 요약 보고

## 에러 핸들링

| 상황 | 전략 |
|------|------|
| 에이전트 1개 실패 | 1회 재시도. 재실패 시 해당 결과 없이 진행 |
| 에이전트 과반 실패 | 사용자에게 알리고 진행 여부 확인 |
```

---

## 작성 원칙

1. 실행 모드를 먼저 명시
2. 에이전트 팀 모드에서는 TeamCreate/SendMessage/TaskCreate 사용법을 구체적으로
3. 파일 경로는 절대적으로 (`_workspace/` 기준)
4. Phase 간 의존성 명시
5. 에러 핸들링은 현실적으로
