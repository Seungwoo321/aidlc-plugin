---
name: aidlc-init
description: AI-DLC 방법론 프로젝트 분석 도구. 프로젝트 정보 수집 → 목적/목표 보고서 생성 → 적용방식 보고서 생성을 수행합니다. Use when user mentions "aidlc", "AI-DLC", "프로젝트 초기화", "방법론 설정", "볼트 사이클", or asks to "initialize project", "setup AI-DLC", "프로젝트 셋팅", "새 프로젝트 시작".
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, AskUserQuestion
metadata:
  author: Seungwoo, Lee
  version: 2.0.0
---

# AI-DLC Initializer

AI-DLC 방법론 프로젝트 분석 도구

<role>
이 스킬은 AI-DLC 방법론을 프로젝트에 적용하기 위한 분석과 보고서 생성을 수행합니다.

핵심 원칙:
- 2단계의 보고서를 순차적으로 생성합니다
- 각 단계에서 사용자의 검토와 피드백을 받습니다
- 참조 백서를 먼저 읽고 이해한 후 실행합니다
</role>

<input_contract>
실행 조건:
- 사용자가 `/aidlc-init` 명령을 실행했을 때

필수 수집 정보:
- 작업 주제 (폴더명에 사용: `docs/aidlc-docs_{주제}/`)
- 프로젝트 상태 (신규 프로젝트 / 리팩토링 / 기능 추가)
- 프로젝트 설명 (목적, 핵심 기능, 타겟 사용자, 기술 스택, 특별 요구사항)
</input_contract>

<output_contract>
최종 산출물:
```
docs/aidlc-docs_{주제}/
├── project-goal-report.md              # 보고서 1: 프로젝트 목적/목표
└── application-approach-report.md      # 보고서 2: AI-DLC 적용방식
```

완료 메시지:
```
AI-DLC 분석이 완료되었습니다.

생성된 보고서:
- docs/aidlc-docs_{주제}/project-goal-report.md
- docs/aidlc-docs_{주제}/application-approach-report.md

다음 단계:
/aidlc-harness를 실행하여 보고서 기반으로 플랜, 폴더 구조, 에이전트/스킬을 생성하세요.
```
</output_contract>

<reference_docs>
실행 전 반드시 다음 백서들을 읽고 이해해야 합니다:

1. **원본 AI-DLC 백서** - `.claude/skills/aidlc-init/docs/ai-dlc-whitepaper-ko.md`
   - Raja SP(AWS)의 AI-DLC 방법론 정의
   - 핵심 철학: 대화 방향의 역전, 볼트 사이클, 계획-승인-실행 패턴

2. **확장 AI-DLC 백서** - `.claude/skills/aidlc-init/docs/ai-dlc-extended-whitepaper.md`
   - 프로젝트 분석 기반 적용 방식
   - 다양한 설계 방법론 지원
</reference_docs>

<execution>
## 실행 플로우

```
1. 스킬 실행 (/aidlc-init)
   ↓
2. 정보 수집 (AI 질문 → 사용자 답변)
   - 작업 주제
   - 프로젝트 상태 (신규/리팩토링/기능추가)
   - 프로젝트 설명
   ↓
3. [보고서 1] 프로젝트 목적/목표 보고서 생성
   → 사용자 검토 & 피드백
   ↓
4. [보고서 2] AI-DLC 적용방식 보고서 생성
   → 사용자 검토 & 피드백
   ↓
5. 완료 → /aidlc-harness로 이어감
```

## 1단계: 정보 수집

**.claude/skills/aidlc-init/templates/01-project-goal.md** 프롬프트의 1단계를 실행합니다.

수집 항목:
- 작업 주제 (폴더명에 사용: `docs/aidlc-docs_{주제}/`)
- 프로젝트 상태
  - 신규 프로젝트: 빈 폴더에서 시작
  - 리팩토링: 기존 프로젝트 개선
  - 기능 추가: 기존 프로젝트에 새 기능 개발
- 프로젝트 설명: 목적, 핵심 기능, 타겟 사용자, 기술 스택, 특별 요구사항

## 2단계: 프로젝트 목적/목표 보고서 생성

**.claude/skills/aidlc-init/templates/01-project-goal.md** 프롬프트의 2단계를 실행합니다.

수집된 정보를 바탕으로 프로젝트의 목적과 목표를 정리한 보고서를 생성합니다.

**산출물:** `docs/aidlc-docs_{주제}/project-goal-report.md`

보고서 구조:
- 산문형 서술로 프로젝트 배경, 목적, 목표 설명
- 표/다이어그램으로 핵심 정보 정리
- 기대 효과 및 성공 기준

사용자 검토 후 승인 또는 피드백을 받습니다.

## 3단계: AI-DLC 적용방식 보고서 생성

**.claude/skills/aidlc-init/templates/02-application-approach.md** 프롬프트를 실행합니다.

프로젝트 특성을 분석하고, 적합한 설계 방법론과 아키텍처 패턴을 선정합니다.

**산출물:** `docs/aidlc-docs_{주제}/application-approach-report.md`

보고서 구조:
- 프로젝트 분석 결과
- 설계 방법론 비교 및 선정 (전문적 근거 포함)
- 아키텍처 패턴 비교 및 선정 (전문적 근거 포함)
- AI-DLC 적용 계획

**중요:** 코드 예시 없이 보고서로서의 역할만 수행합니다.

사용자 검토 후 승인 또는 피드백을 받습니다.
</execution>

<file_structure>
```
aidlc-plugin/
├── skills/
│   └── aidlc-init/
│       ├── SKILL.md                       # 스킬 정의
│       ├── docs/
│       │   ├── ai-dlc-whitepaper-ko.md        # 원본 AI-DLC 백서
│       │   └── ai-dlc-extended-whitepaper.md  # 확장 AI-DLC 백서
│       └── templates/
│           ├── 01-project-goal.md             # 정보 수집 + 보고서 1 생성
│           └── 02-application-approach.md     # 보고서 2 생성
```
</file_structure>

<constraints>
DO:
- 참조 백서를 먼저 읽고 이해한 후 실행합니다
- 각 단계에서 사용자의 검토와 피드백을 받습니다
- 보고서 생성 후 반드시 사용자 승인을 받고 다음 단계로 진행합니다
- 기존 프로젝트(리팩토링/기능추가)인 경우 설정 파일, 디렉토리 구조 분석을 수행합니다

DO NOT:
- 사용자 승인 없이 다음 단계로 넘어가지 않습니다
- 적용방식 보고서에 코드 예시를 포함하지 않습니다
- 기존 `docs/aidlc-docs_{주제}/` 폴더가 있을 때 확인 없이 덮어쓰지 않습니다
- 폴더 구조, plan.md, 에이전트/스킬을 생성하지 않습니다 — 이는 `/aidlc-harness`의 역할입니다
</constraints>

<critical>
- 각 보고서는 반드시 사용자 검토와 승인을 거쳐야 다음 단계로 진행합니다
- 참조 백서의 AI-DLC 철학(대화 방향의 역전, 볼트 사이클, 계획-승인-실행 패턴)을 준수합니다
</critical>
