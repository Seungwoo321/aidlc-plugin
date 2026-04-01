---
name: aidlc-harness
description: AIDLC 프로젝트의 도메인 에이전트와 스킬을 생성하는 하네스. aidlc-init 산출물(보고서 2개)을 기반으로 프로젝트 특성에 맞는 전문 에이전트를 설계하고, 에이전트가 사용할 스킬을 생성합니다. Use when user mentions "하네스", "harness", "에이전트 생성", "도메인 에이전트", or asks to "generate agents", "create harness", "에이전트 만들어줘", "하네스 구성". /aidlc-init 완료 후 실행.
user-invocable: true
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, AskUserQuestion
metadata:
  author: Seungwoo Lee
  version: 1.0.0
  based-on: revfactory/harness (Apache-2.0)
---

# AIDLC Harness

aidlc-init 산출물을 기반으로 프로젝트 도메인에 최적화된 에이전트와 스킬을 생성한다.

<role>
이 스킬은 AI-DLC 방법론의 두 번째 단계로, 프로젝트 보고서를 분석하여 도메인 특화 에이전트 팀을 설계하고 생성한다.

핵심 원칙:
- aidlc-init의 보고서를 입력으로 사용하여 중복 질문을 하지 않는다
- 고정 에이전트(thinking-partner, genius-thinker)는 플러그인에 내장되어 있으므로 생성하지 않는다
- 프로젝트 도메인에 특화된 에이전트만 생성한다
- 모든 에이전트는 `.claude/agents/{name}.md` 파일로 정의한다
</role>

<input_contract>
전제 조건:
- `/aidlc-init`이 완료되어 있어야 한다
- `docs/aidlc-docs_{주제}/project-goal-report.md` 존재
- `docs/aidlc-docs_{주제}/application-approach-report.md` 존재

인수:
- 프로젝트 경로 (없으면 cwd)
- 특정 AIDLC docs 경로 (여러 주제가 있을 경우)
</input_contract>

<output_contract>
산출물:
```
프로젝트/
├── .claude/
│   ├── agents/
│   │   ├── {domain-agent-1}.md    # 도메인 에이전트 정의
│   │   ├── {domain-agent-2}.md
│   │   └── ...
│   └── skills/
│       └── {domain-skill}/        # 도메인 스킬 (필요 시)
│           ├── skill.md
│           └── references/
└── docs/aidlc-docs_{주제}/
    └── harness-report.md          # 하네스 구성 보고서
```
</output_contract>

<execution>
## 실행 플로우

```
1. AIDLC 보고서 읽기
   ↓
2. Phase 1: 도메인 분석
   ↓
3. Phase 2: 팀 아키텍처 설계
   → 사용자 승인
   ↓
4. Phase 3: 에이전트 정의 생성
   → 사용자 검토
   ↓
5. Phase 4: 도메인 스킬 생성 (필요 시)
   ↓
6. Phase 5: 오케스트레이션 설정
   ↓
7. Phase 6: 검증
   ↓
8. 하네스 보고서 생성
```

---

## Phase 1: 도메인 분석

### 1-1. AIDLC 보고서 수집

다음 파일들을 읽는다:
- `docs/aidlc-docs_{주제}/project-goal-report.md`
- `docs/aidlc-docs_{주제}/application-approach-report.md`
- `docs/aidlc-docs_{주제}/plan.md` (있는 경우)
- `CLAUDE.md` (프로젝트 지침)

### 1-2. 기존 에이전트/스킬 확인

프로젝트에 이미 존재하는 에이전트와 스킬을 스캔한다:
- `.claude/agents/*.md`
- `.claude/skills/*/`
- 충돌 가능성을 파악한다

### 1-3. 코드베이스 탐색 (기존 프로젝트인 경우)

- `package.json`, `tsconfig.json`, `pyproject.toml` 등 설정 파일
- 디렉토리 구조 분석
- 주요 모듈/패키지 파악
- 기술 스택 확인

### 1-4. 핵심 작업 유형 식별

보고서에서 다음을 추출한다:
- 프로젝트가 다루는 도메인 영역
- 핵심 작업 유형 (UI 개발, API 개발, 데이터 처리, 알고리즘 등)
- 프로젝트 단계별 필요 역할
- 특수한 도메인 지식 요구사항

---

## Phase 2: 팀 아키텍처 설계

### 2-1. 실행 모드 선택

`references/architecture-patterns.md`의 의사결정 트리를 참조하여 선택한다.

| 모드 | 조건 |
|------|------|
| 에이전트 팀 (기본) | 에이전트 2개 이상 + 상호 통신 필요 |
| 서브 에이전트 | 에이전트 간 통신 불필요 또는 단일 에이전트 |

### 2-2. 아키텍처 패턴 선택

`references/architecture-patterns.md`의 6가지 패턴에서 선택한다:

1. **Pipeline** — 순차적 단계 의존
2. **Fan-out/Fan-in** — 병렬 처리 후 통합
3. **Expert Pool** — 상황별 전문가 선택
4. **Producer-Reviewer** — 생성-검증 쌍
5. **Supervisor** — 동적 작업 분배
6. **Hierarchical** — 계층적 위임 (2단계 이내)

복합 패턴도 고려한다 (예: Pipeline + Fan-out, Fan-out + Producer-Reviewer).

### 2-3. 에이전트 분리 기준

| 기준 | 분리 | 통합 |
|------|------|------|
| 전문성 | 영역이 다르면 분리 | 영역이 겹치면 통합 |
| 병렬성 | 독립 실행 가능하면 분리 | 순차 종속이면 통합 고려 |
| 컨텍스트 | 컨텍스트 부담이 크면 분리 | 가볍고 빠르면 통합 |
| 재사용성 | 다른 팀에서도 쓰면 분리 | 이 팀에서만 쓰면 통합 고려 |

### 2-4. 고정 에이전트와의 관계

플러그인에 내장된 고정 에이전트:
- **thinking-partner**: 구조 분석, 방향성 논의, 의사결정 지원
- **genius-thinker**: 혁신적 아이디어 발상, 문제 재정의

이 두 에이전트는 생성하지 않는다. 도메인 에이전트가 이들과 어떻게 협업할지를 설계한다.

### 2-5. 사용자 승인

AskUserQuestion으로 팀 아키텍처를 제안한다:
```
아키텍처 패턴: {선택한 패턴}
실행 모드: {에이전트 팀 / 서브 에이전트}

제안하는 도메인 에이전트:
1. {agent-name} — {역할 설명}
2. {agent-name} — {역할 설명}
...

고정 에이전트 활용 계획:
- thinking-partner: {어떤 단계에서 활용}
- genius-thinker: {어떤 단계에서 활용}

이 구성으로 진행할까요?
```

---

## Phase 3: 에이전트 정의 생성

승인된 구성에 따라 `.claude/agents/{name}.md` 파일을 생성한다.

### 에이전트 정의 필수 구조

`references/architecture-patterns.md`의 에이전트 정의 구조를 따른다:

```markdown
---
name: {agent-name}
description: "{1-2문장 역할 설명. 트리거 키워드 나열.}"
tools: {필요한 도구 목록}
model: opus
---

# {Agent Name} — 역할 한줄 요약

당신은 {도메인}의 {역할} 전문가입니다.

## 핵심 역할
1. 역할1
2. 역할2

## AI-DLC 원칙 준수
- **대화 방향의 역전**: 먼저 분석/제안하고 승인받기
- **계획-승인-실행 패턴**: 계획 → 승인 → 실행
- **볼트 사이클**: 볼트 범위 내에서 완결

## 작업 원칙
- 원칙1 (Why: 이유)
- 원칙2 (Why: 이유)

## 입력/출력 프로토콜
- 입력: [어디서 무엇을 받는지]
- 출력: [어디에 무엇을 쓰는지]
- 형식: [파일 포맷, 구조]

## 팀 통신 프로토콜
- 메시지 수신: [누구로부터 어떤 메시지를 받는지]
- 메시지 발신: [누구에게 어떤 메시지를 보내는지]
- 작업 요청: [공유 작업 목록에서 어떤 유형의 작업을 요청하는지]

## 에러 핸들링
- [실패 시 행동]
- [타임아웃 시 행동]

## 협업
- 다른 에이전트와의 관계
```

### 작성 원칙

- **Why-First**: 규칙보다 이유를 설명한다. LLM은 이유를 이해하면 엣지 케이스에서도 올바르게 판단한다
- **관계 파악 원칙**: 기존 코드/인프라와의 관계를 먼저 파악하라
- **확장 원칙**: 사용자의 기존 코드는 의사결정의 결과물이다. 재설계를 제안하지 말고 기존 설계 위에 확장하라
- **모든 에이전트 `model: "opus"`**: Agent 도구 호출 시 반드시 명시

---

## Phase 4: 도메인 스킬 생성 (필요 시)

에이전트가 반복적으로 사용할 절차적 지식이 있으면 스킬로 분리한다.

### 스킬 vs 에이전트 구분

| 구분 | 스킬 (Skill) | 에이전트 (Agent) |
|------|-------------|-----------------|
| 정의 | 절차적 지식 + 도구 번들 | 전문가 페르소나 + 행동 원칙 |
| 위치 | `.claude/skills/` | `.claude/agents/` |
| 용도 | "어떻게 하는가" | "누가 하는가" |

### Progressive Disclosure 적용

`references/skill-writing-guide.md`를 참조한다:

- 스킬 본문 < 500줄. 초과 시 `references/`로 분리
- description은 "pushy"하게 작성 — 트리거 상황을 구체적으로 명시
- 조건부 로드: 큰 reference는 필요할 때만 Read로 로드

### 스킬 생성 시 참고

- `references/skill-writing-guide.md` — 스킬 작성법
- `references/qa-agent-guide.md` — QA 에이전트가 필요한 경우

---

## Phase 5: 오케스트레이션 설정

`references/orchestrator-template.md`를 참조하여, aidlc-team이 생성된 에이전트를 활용할 수 있도록 설정한다.

### 하네스 보고서 생성

`docs/aidlc-docs_{주제}/harness-report.md`에 다음을 기록한다:

```markdown
# 하네스 구성 보고서

## 프로젝트 도메인 분석 요약
{도메인 특성, 핵심 작업 유형}

## 아키텍처
- 실행 모드: {에이전트 팀 / 서브 에이전트}
- 패턴: {선택한 패턴}

## 에이전트 구성

### 고정 에이전트 (플러그인 내장)
- thinking-partner — {이 프로젝트에서의 활용}
- genius-thinker — {이 프로젝트에서의 활용}

### 도메인 에이전트 (이 프로젝트용 생성)
| 에이전트 | 파일 | 역할 | 주 활용 단계 |
|----------|------|------|-------------|
| {name} | `.claude/agents/{name}.md` | {역할} | {Inception/Construction/Operations} |
| ... | | | |

## 도메인 스킬 (생성된 경우)
| 스킬 | 경로 | 용도 |
|------|------|------|
| {name} | `.claude/skills/{name}/` | {용도} |

## aidlc-team 연동
- 팀 구성 시 참조할 에이전트 목록
- 단계별 권장 팀 구성
- 에이전트 간 통신 프로토콜 요약
```

---

## Phase 6: 검증

### 6-1. 구조 검증

- [ ] 모든 에이전트 파일이 `.claude/agents/`에 존재
- [ ] 에이전트 정의에 필수 섹션(역할, 원칙, I/O, 통신, 에러)이 포함
- [ ] 모든 에이전트에 `model: opus` 명시
- [ ] 고정 에이전트(thinking-partner, genius-thinker)를 중복 생성하지 않음
- [ ] 스킬이 있으면 description이 구체적이고 트리거 조건이 명확

### 6-2. 통신 프로토콜 검증

- [ ] 에이전트 간 SendMessage 대상이 서로 일치
- [ ] 입출력 파일 경로가 충돌하지 않음
- [ ] 의존성이 순환하지 않음

### 6-3. aidlc-team 연동 검증

- [ ] harness-report.md가 생성됨
- [ ] aidlc-team이 생성된 에이전트를 스캔하여 팀을 구성할 수 있는 정보가 충분
</execution>

<constraints>
DO:
- aidlc-init 보고서를 입력으로 활용하여 중복 질문 최소화
- 에이전트 정의에 Why-First 원칙 적용
- 프로젝트 도메인에 특화된 역할만 생성
- 사용자 승인 후에만 파일 생성

DO NOT:
- thinking-partner, genius-thinker를 중복 생성하지 않는다
- 범용 에이전트(implementer, spec-auditor 등)를 그대로 복사하지 않는다 — 도메인 맥락을 반영한 특화 버전을 만든다
- 에이전트 역할을 Agent 도구 프롬프트에 직접 넣지 않는다 — 반드시 파일로 정의
- 코드 예시를 보고서에 포함하지 않는다
</constraints>

<critical>
- 모든 에이전트는 반드시 `.claude/agents/{name}.md` 파일로 정의한다. 파일로 존재해야 다음 세션에서 재사용 가능하다
- 에이전트 정의에 팀 통신 프로토콜을 반드시 포함한다. 협업 품질은 통신 프로토콜의 명확성에 달려있다
- Phase 2에서 사용자 승인 없이 Phase 3으로 넘어가지 않는다
</critical>
