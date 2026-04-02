<!--
  Copyright (c) revfactory (https://github.com/revfactory/harness)
  Licensed under the Apache License, Version 2.0
  https://www.apache.org/licenses/LICENSE-2.0

  Modified by Seungwoo Lee for AIDLC Plugin.
  Changes: AIDLC 방법론 맥락에 맞게 수정, 요약, AIDLC 프로젝트 예시 추가
-->

# 스킬 작성 가이드

> 출처: revfactory/harness (Apache-2.0)

하네스에서 생성하는 스킬의 품질을 높이기 위한 상세 작성 가이드.

---

## 1. Description 작성 패턴

Description은 스킬의 유일한 트리거 메커니즘이다. Claude는 `available_skills` 목록에서 name + description만 보고 스킬 사용 여부를 결정한다.

### 트리거 메커니즘 이해

Claude는 단순 작업에는 스킬을 호출하지 않는 경향이 있다. 복잡하고 다단계이며 전문적인 작업일수록 스킬 트리거 확률이 높다.

### 작성 원칙

1. **스킬이 하는 일** + **구체적 트리거 상황**을 모두 기술
2. 유사하지만 트리거하면 안 되는 경우를 구분하는 경계 조건 명시
3. 약간 "pushy"하게 — Claude의 보수적 판단을 보상

### 좋은 예시

```yaml
description: "PDF 파일 읽기, 텍스트/테이블 추출, 병합, 분할, 회전, 워터마크,
  암호화/복호화, OCR 등 모든 PDF 작업을 수행. .pdf 파일을 언급하거나
  PDF 산출물을 요청하면 반드시 이 스킬을 사용할 것."
```

---

## 2. 본문 작성 스타일

### Why-First 원칙

LLM은 이유를 이해하면 엣지 케이스에서도 올바르게 판단한다.

**나쁜 예:**
```markdown
ALWAYS use pdfplumber for table extraction. NEVER use PyPDF2 for tables.
```

**좋은 예:**
```markdown
테이블 추출에는 pdfplumber를 사용한다. PyPDF2는 텍스트 추출에 특화되어
있어 테이블의 행/열 구조를 보존하지 못하기 때문이다.
```

### 일반화 원칙

피드백에서 문제가 발견되면, 특정 예시에만 맞는 좁은 수정 대신 원리 수준에서 일반화한다.

### 컨텍스트 절약

모든 문장이 토큰 비용을 정당화하는지 자문한다:
- "Claude가 이미 알고 있는 내용인가?" → 삭제
- "이 설명이 없으면 Claude가 실수하는가?" → 유지
- "구체적 예시 하나가 긴 설명보다 효과적인가?" → 예시로 대체

---

## 3. Progressive Disclosure 패턴

### 패턴 1: 도메인별 분리

```
skill/
├── skill.md (개요 + 도메인 선택 가이드)
└── references/
    ├── finance.md
    ├── sales.md
    └── product.md
```

사용자가 매출에 대해 물으면 finance.md만 로드.

### 패턴 2: 조건부 상세

```markdown
## 문서 생성
docx-js로 새 문서를 생성한다. → [DOCX-JS.md](references/docx-js.md) 참조.

## 문서 편집
단순 편집은 XML을 직접 수정.
**추적 변경이 필요하면**: [REDLINING.md](references/redlining.md) 참조
```

### 패턴 3: 대형 레퍼런스

300줄 이상의 reference 파일은 상단에 목차를 포함한다.

---

## 4. 스크립트 번들링 판단 기준

| 신호 | 조치 |
|------|------|
| 3개 테스트 중 3개에서 동일한 헬퍼 스크립트 생성 | `scripts/`에 번들링 |
| 매번 같은 pip install/npm install 실행 | 스킬에 의존성 설치 단계 명시 |
| 동일한 다단계 접근법 반복 | 스킬 본문에 표준 절차로 기술 |
| 매번 비슷한 에러 후 같은 회피책 적용 | 스킬에 알려진 문제와 해결법 기술 |

---

## 5. 스킬에 포함하지 않을 것

- README.md, CHANGELOG.md 등 부가 문서
- 스킬 생성 과정의 메타 정보
- 사용자 대상 설명서 (스킬은 AI 에이전트를 위한 지시서)
- Claude가 이미 알고 있는 일반적 지식
