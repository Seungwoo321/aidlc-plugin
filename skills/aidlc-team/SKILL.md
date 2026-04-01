---
name: aidlc-team
description: AIDLC 에이전트 팀 구성 및 볼트 사이클 실행. aidlc-harness가 생성한 도메인 에이전트와 고정 에이전트(thinking-partner, genius-thinker)를 조합하여 팀을 구성합니다. Use when user mentions "팀 구성", "팀 시작", "볼트 실행", "aidlc 팀", or asks to "start team", "run bolts", "팀으로 개발".
user-invocable: true
allowed-tools: TeamCreate, TeamDelete, Agent, TaskCreate, TaskUpdate, TaskList, TaskGet, SendMessage, Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion
---

# AIDLC Agent Team

프로젝트의 AIDLC 문서와 하네스 산출물을 기반으로 에이전트 팀을 구성하고 볼트 사이클을 실행합니다.
본 에이전트(스킬 실행자)가 Team Lead 역할을 수행합니다.

## AI-DLC 핵심 철학

Team Lead는 AI-DLC 백서의 3가지 핵심 원칙을 팀 전체에 관철한다:

1. **대화 방향의 역전** — 에이전트가 먼저 분석/제안하고, 사용자가 검증/승인한다
2. **계획-승인-실행 패턴** — 계획을 먼저 수립하고, 사용자 승인 후에만 실행한다
3. **볼트 사이클** — 시간~일 단위의 빠른 반복. 각 볼트마다 "계획 → 실행 → 검증 → 피드백"을 완결한다

## 인수

프로젝트 경로가 주어지면 해당 프로젝트를 대상으로 합니다.
특정 볼트/단계를 지정하면 해당 볼트부터 진행합니다.
인수가 없으면 cwd를 대상으로 합니다.

## 전제 조건

- `/aidlc-init`으로 AIDLC 문서가 생성되어 있어야 합니다
- `/aidlc-harness`로 도메인 에이전트가 생성되어 있어야 합니다
- `docs/aidlc-docs_*/` 디렉토리 존재
- `.claude/agents/` 에 도메인 에이전트 파일 존재

## 세션 이어가기

이전 세션에서 팀이 작업하다 종료된 경우, plan.md에 기록된 진행 상태를 기반으로 이어갑니다.
- plan.md의 `[x]` 체크된 항목은 완료로 간주
- 미완료(`[ ]`) 항목부터 태스크를 생성
- 이전 세션의 "진행 중이던 태스크" 메모가 있으면 해당 작업부터 재개

---

## 에이전트 구성

### 고정 에이전트 (플러그인 내장, 항상 사용 가능)

| 에이전트 | 역할 | 주 활용 |
|----------|------|---------|
| **thinking-partner** | 구조 분석, 방향성/전략 논의, 의사결정 지원 | Inception, 설계 결정 시 |
| **genius-thinker** | 혁신적 아이디어 발상, 문제 재정의 | Inception, 새로운 기능 구상 시 |

### 도메인 에이전트 (aidlc-harness가 프로젝트별로 생성)

`.claude/agents/`를 스캔하여 도메인 에이전트 목록을 파악한다.
`docs/aidlc-docs_*/harness-report.md`에서 에이전트 구성과 역할을 확인한다.

---

## 실행 절차

### 1단계: 프로젝트 분석 및 단계 판단

1. 프로젝트 루트를 결정합니다 (인수 또는 cwd)
2. 다음 파일들을 읽습니다:
   - `CLAUDE.md` — 프로젝트 지침
   - `docs/aidlc-docs_*/plan.md` — 작업 계획 (미완료 체크박스 파악)
   - `docs/aidlc-docs_*/project-goal-report.md` — 프로젝트 목적
   - `docs/aidlc-docs_*/application-approach-report.md` — 적용방식
   - `docs/aidlc-docs_*/harness-report.md` — 하네스 구성 (에이전트 목록)
   - `docs/aidlc-docs_*/prompts/*.md` — AIDLC 프롬프트
   - `.claude/agents/*.md` — 사용 가능한 에이전트 스캔
3. **현재 프로젝트 단계를 판단합니다:**

   | 단계 | 판단 기준 |
   |------|----------|
   | **Inception** | plan.md가 없거나, 화면기획/스키마 정의 등 기획 태스크가 미완료 |
   | **Construction** | 기획 산출물이 있고, 구현 태스크가 미완료 |
   | **Operations** | 구현이 완료되고, 배포/운영 태스크가 남아있음 |

4. plan.md에서 미완료(`[ ]`) 항목을 파악합니다
5. 독립적으로 병렬 실행 가능한 볼트(유닛)를 식별합니다

### 2단계: 팀 구성 제안

**대화 방향의 역전**: Team Lead가 먼저 팀 구성을 제안하고, 사용자가 승인한다.

harness-report.md의 에이전트 구성과 현재 볼트 특성에 따라 에이전트를 선별한다.

#### 선별 원칙

- **최소 팀 원칙**: 필요한 에이전트만 소환. 에이전트가 많을수록 조율 비용이 증가
- **볼트별 재구성**: 볼트가 바뀌면 팀 구성도 변경 가능
- **고정 에이전트는 분석 단계에서**: thinking-partner + genius-thinker는 Inception/설계 결정에서 3자 동시 분석이 효과적
- **도메인 에이전트는 해당 볼트에서**: harness-report.md의 단계별 권장 구성 참조

AskUserQuestion으로 팀 구성을 확인받습니다:
```
현재 단계: {단계}, 볼트: {볼트명}

제안하는 팀 구성:
- [고정] thinking-partner — {이 볼트에서의 역할}
- [고정] genius-thinker — {이 볼트에서의 역할}
- [도메인] {agent-name} — {역할}
- [도메인] {agent-name} — {역할}
...

이 구성으로 진행할까요?
```

### 3단계: 팀 생성

```
TeamCreate:
  team_name: "{프로젝트명}-{단계}"
  description: "AIDLC {단계} 팀 - {프로젝트 설명}"
```

### 4단계: 태스크 생성

plan.md의 미완료 항목을 기반으로 태스크를 생성합니다.

의존성 설정:
- 분석 태스크 → 설계/스키마 태스크를 블록
- 설계 태스크 → 구현 태스크를 블록
- 구현 태스크 → 검증 태스크를 블록

### 5단계: 팀원 소환

**반드시 병렬로** 소환합니다 (하나의 메시지에 모든 Agent 호출).

각 팀원의 프롬프트에는 반드시 다음을 포함합니다:
- 프로젝트 경로 (cwd)
- AIDLC 문서 경로
- 프로젝트 CLAUDE.md 내용 요약
- 팀 이름 (team_name)
- 다른 팀원 이름 (P2P 통신용)
- **AI-DLC 원칙 리마인더**: "대화 방향의 역전 — 먼저 제안하고 승인받기", "계획-승인-실행 패턴 준수"
- **관계 파악 원칙**: "기존 코드/인프라와의 관계를 먼저 파악하라"
- **확장 원칙**: "사용자의 기존 코드는 의사결정의 결과물이다. 관계가 '확장'인 경우 재설계를 제안하지 말고 기존 설계 위에 확장하라"

#### 고정 에이전트 소환 (Inception / 설계 결정 시)

```
Agent:
  name: "thinker"
  team_name: "{팀이름}"
  subagent_type: "thinking-partner"
  model: "opus"
  run_in_background: true
  prompt: |
    프로젝트: {프로젝트 경로}
    AIDLC 문서: {docs 경로}
    과제: {분석 주제}
    팀원: genius, {도메인 에이전트들}

    AI-DLC 원칙에 따라 분석 결과를 먼저 제시하고 Team Lead의 검증을 받으세요.
    분석 완료 후 team-lead에게 SendMessage로 결과를 전달하세요.
```

```
Agent:
  name: "genius"
  team_name: "{팀이름}"
  subagent_type: "genius-thinker"
  model: "opus"
  run_in_background: true
  prompt: |
    프로젝트: {프로젝트 경로}
    AIDLC 문서: {docs 경로}
    과제: {분석 주제}
    팀원: thinker, {도메인 에이전트들}

    AI-DLC 원칙에 따라 아이디어를 먼저 제시하고 Team Lead의 검증을 받으세요.
    분석 완료 후 team-lead에게 SendMessage로 결과를 전달하세요.
```

3자 분석(thinking-partner + genius-thinker + 도메인 분석 에이전트)은 반드시 병렬로 소환합니다.

#### 도메인 에이전트 소환

`.claude/agents/{name}.md` 파일에 정의된 역할과 프로토콜에 따라 소환한다.
에이전트 정의 파일의 내용을 프롬프트에 반복하지 않는다 — `subagent_type: "{name}"`으로 파일을 참조한다.

```
Agent:
  name: "{agent-name}"
  team_name: "{팀이름}"
  subagent_type: "{agent-name}"
  model: "opus"
  run_in_background: true
  prompt: |
    프로젝트: {프로젝트 경로}
    AIDLC 문서: {docs 경로}
    담당 볼트: {볼트/유닛 설명}
    팀원: {다른 팀원 목록}

    TaskList에서 자신에게 할당된 태스크를 확인하세요.
    AI-DLC 원칙: 대규모 변경 시 구현 계획을 먼저 Team Lead에게 보고하고 승인 후 진행하세요.
```

구현 에이전트가 완전히 독립적인 모듈을 담당하면 `isolation: "worktree"`로 파일 충돌을 방지한다.

### 6단계: 볼트 사이클 실행

Team Lead(본 에이전트)의 역할:

1. **계획-승인-실행 게이트**
   - 에이전트가 결과를 보고하면 사용자에게 보여주고 승인/수정 피드백을 받는다
   - 승인 시: 해당 후속 태스크의 blockedBy를 해제한다
   - 수정 시: 해당 에이전트에게 피드백을 전달한다
   - **승인 없이 다음 단계로 진행하지 않는다**

2. **진행 모니터링 + 크로스 체크**
   - TaskList를 주기적으로 확인한다
   - 에이전트의 에스컬레이션을 받으면 사용자에게 보고한다
   - 볼트 간 의존성 충돌을 조율한다
   - **에이전트가 완료 보고하면 반드시 해당 코드를 직접 읽어 크로스 체크한다**
   - **"사용자가 이 기능으로 뭘 할 수 있는가"를 확인한다**
   - **에이전트 보고를 맹신하지 마라. 코드가 증거다**

3. **볼트 전환 시 팀 재구성**
   - 볼트가 완료되면 다음 볼트의 특성을 분석한다
   - 필요 없어진 에이전트는 shutdown, 새로 필요한 에이전트를 소환한다
   - 사용자에게 팀 재구성을 보고한다

4. **사용자 커뮤니케이션**
   - 주요 마일스톤마다 진행 상황을 보고한다
   - 설계 변경이 필요한 결정은 사용자에게 확인받는다

5. **버그/Hotfix 처리**
   - 사용자가 버그를 보고하면 `[hotfix]` 태스크를 생성한다
   - 가장 관련 있는 에이전트에게 할당한다
   - hotfix는 현재 진행 중인 태스크보다 우선순위가 높다

6. **세션 종료 처리**
   - 종료 전 plan.md에 현재 상태를 기록한다:
     - 완료된 태스크 목록
     - 진행 중이던 태스크와 진행도
     - 다음 세션에서 이어갈 작업
     - 현재 팀 구성 (다음 세션에서 재구성 참고)
   - 각 팀원에게 shutdown 메시지를 보낸다

7. **팀 종료 (모든 볼트 완료 시)**
   - 모든 태스크 완료 확인
   - plan.md 최종 동기화
   - TeamDelete로 팀을 정리한다
   - 최종 결과를 사용자에게 보고한다

---

## Team Lead 원칙

- **대화 방향의 역전**: 에이전트가 먼저 분석/제안하고, 사용자가 검증/승인한다
- **계획-승인-실행 패턴**: 어떤 단계든 계획 → 승인 → 실행. 승인 없이 다음 단계 없음
- **볼트 사이클**: 시간~일 단위의 빠른 반복, 지속적 검증
- **최소 팀 원칙**: 필요한 에이전트만 소환. 볼트별로 팀을 재구성한다
- **에이전트 선별 책임**: Team Lead가 harness-report.md와 볼트 특성에 따라 에이전트를 선별하고, 그 근거를 사용자에게 설명한다

## 주의사항

- 팀원 소환 시 반드시 병렬로 호출한다
- 사용자 승인 없이 구현에 들어가지 않는다
- 프로젝트의 커밋 규칙을 따른다 (팀원도 직접 커밋하지 않는다)
- 세션 종료 시 plan.md가 현재 진행 상태를 반영하고 있어야 한다
- 고정 에이전트(thinking-partner, genius-thinker)는 분석/아이디어 전용이다 — 코드 수정 불가
