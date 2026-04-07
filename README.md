# AIDLC Plugin

AI-DLC 방법론을 Claude Code에 적용하는 플러그인입니다. 프로젝트 분석부터 도메인 에이전트 생성, 볼트 사이클 실행까지 하나의 파이프라인으로 제공합니다.

## 파이프라인

```
/aidlc-init → /aidlc-harness → /aidlc-run
```

| 단계 | 스킬 | 역할 | 설명 |
|------|------|------|------|
| 1 | `/aidlc-init` | 분석 (What) | 프로젝트 정보 수집 → 목적/목표 보고서 → 적용방식 보고서 생성 |
| 2 | `/aidlc-harness` | 설계+구성 (How) | 작업 계획(plan.md), 폴더 구조, 도메인 에이전트/스킬 생성 |
| 3 | `/aidlc-run` | 실행 (Do) | plan.md 기반 볼트 사이클 실행 (에이전트 팀 또는 서브 에이전트) |

## 내장 에이전트

플러그인에 내장된 고정 에이전트로, `/aidlc-harness`에서 생성하지 않습니다.

| 에이전트 | 역할 |
|----------|------|
| `thinking-partner` | 사고확장 파트너. 기획/방향성/전략 논의 시 더 깊은 구조를 이해하도록 돕는다 |
| `genius-thinker` | 천재적 사고 확장. 10가지 사고 공식 기반으로 최소 10개 이상의 아이디어를 도출한다 |

## 구조

```
# 플러그인 구조
.claude-plugin/
  plugin.json
  marketplace.json
agents/
  thinking-partner.md
  genius-thinker.md
skills/
  aidlc-init/
    SKILL.md
    docs/                    # AI-DLC 백서
    templates/               # 보고서 생성 템플릿
  aidlc-harness/
    SKILL.md
    references/              # 아키텍처 패턴, 스킬 작성 가이드 등
  aidlc-run/
    SKILL.md
```

### 산출물 구조 (대상 프로젝트에 생성됨)

```
docs/aidlc-docs_{주제}/
  project-goal-report.md          # aidlc-init이 생성
  application-approach-report.md  # aidlc-init이 생성
  plan.md                         # aidlc-harness가 생성
  harness-report.md               # aidlc-harness가 생성
  inception/                      # aidlc-harness가 생성
  construction/
  operations/
.claude/agents/                   # aidlc-harness가 생성 (도메인 에이전트)
.claude/skills/                   # aidlc-harness가 생성 (필요 시)
```

## AI-DLC 핵심 원칙

1. **대화 방향의 역전** - 에이전트가 먼저 분석/제안하고, 사용자가 검증/승인한다
2. **계획-승인-실행 패턴** - 계획을 먼저 수립하고, 사용자 승인 후에만 실행한다
3. **볼트 사이클** - 시간~일 단위의 빠른 반복. 각 볼트마다 "계획 -> 실행 -> 검증 -> 피드백"을 완결한다

## Acknowledgments

`aidlc-harness` 스킬의 참조 자료는 [revfactory/harness](https://github.com/revfactory/harness) (Apache-2.0)를 기반으로 수정되었습니다. 자세한 내용은 [NOTICE](./NOTICE)를 참조하세요.

## License

[MIT](./LICENSE)
