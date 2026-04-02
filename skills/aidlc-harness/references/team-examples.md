<!--
  Copyright (c) revfactory (https://github.com/revfactory/harness)
  Licensed under the Apache License, Version 2.0
  https://www.apache.org/licenses/LICENSE-2.0

  Modified by Seungwoo Lee for AIDLC Plugin.
  Changes: AIDLC 방법론 맥락에 맞게 수정, 요약, AIDLC 프로젝트 예시 추가
-->

# 에이전트 팀 구성 예시

> 출처: revfactory/harness (Apache-2.0), AIDLC 맥락 요약

---

## 1. 리서치 팀 (Fan-out/Fan-in, 에이전트 팀)

```
[리더] → TeamCreate → [공식조사] ←→ [미디어조사] ←→ [커뮤니티조사] ←→ [배경조사]
                          ↓              ↓              ↓              ↓
                    공식 데이터      미디어 트렌드   커뮤니티 반응    경쟁 배경
                          └──────────── Read ───────────────────────┘
                                        ↓
                                  [리더: 통합 보고서]
```

**핵심:** 조사자 간 발견을 SendMessage로 실시간 공유. 한 에이전트의 발견이 다른 에이전트의 조사 방향을 수정할 수 있어 단독 조사 대비 품질 향상.

---

## 2. 소설 집필 팀 (Pipeline + Fan-out)

```
Phase 1 (병렬): [세계관] + [캐릭터] + [플롯]
Phase 2 (순차): [집필]
Phase 3 (병렬): [과학검증] + [연속성검증]
```

Phase 1에서 3자가 병렬로 설정을 구축하고, 서로의 설정을 참조하며 일관성 유지. Phase 2에서 집필, Phase 3에서 병렬 검증.

---

## 3. 웹툰 제작 팀 (Producer-Reviewer, 서브 에이전트)

```
[오케스트레이터] → [아티스트] → 패널 생성
                → [리뷰어] → PASS/FIX/REDO 판정
                              ↓ FIX/REDO
                → [아티스트] → 수정 (최대 2회)
```

**판정 기준:**
- PASS: 그대로 사용
- FIX: 구체적 수정 지시와 함께 아티스트에게 반환
- REDO: 전체 재생성

---

## 4. 코드 리뷰 팀 (Fan-out/Fan-in, 에이전트 팀)

```
[리더] → [보안리뷰어] ←→ [성능리뷰어] ←→ [테스트리뷰어]
```

**핵심:** "다른 관점을 가진 에이전트들이 계층 필터링 없이 직접 발견을 공유." 보안 이슈가 성능에 영향을 주는 교차 도메인 이슈를 빠르게 감지.

---

## 5. 코드 마이그레이션 팀 (Supervisor, 에이전트 팀)

```
[마이그레이션 감독자] → 파일 배치 분석 → 작업 할당
                     → [마이그레이터A] → 배치 1 처리
                     → [마이그레이터B] → 배치 2 처리
                     → [마이그레이터C] → 배치 3 처리
```

감독자가 공유 작업 목록(TaskCreate)으로 작업 등록. 마이그레이터들이 자체 요청으로 작업 수행. 완료 속도에 따라 감독자가 동적 재할당.

---

## AIDLC 프로젝트 팀 구성 예시

### 웹앱 프로젝트 (예: StarYam)

```
고정: thinking-partner, genius-thinker

도메인 에이전트:
- saju-content-dev   — 사주/운세 콘텐츠 로직 구현
- frontend-dev       — React/Next.js UI 구현
- api-dev            — API 엔드포인트 구현
- qa-inspector       — 통합 정합성 검증

패턴: Fan-out (Inception) → Pipeline + Fan-out (Construction)
```

### DSL/파서 프로젝트 (예: Wireweave)

```
고정: thinking-partner, genius-thinker

도메인 에이전트:
- dsl-parser-dev     — DSL 문법 파서 구현
- renderer-dev       — HTML/CSS 렌더링 구현
- schema-validator   — 스키마 검증 + 하위 호환성
- doc-writer         — API 문서 + 사용 가이드

패턴: Pipeline (파서 → 렌더러) + Producer-Reviewer (구현 → 검증)
```

### 데스크톱 앱 프로젝트 (예: Pengent)

```
고정: thinking-partner, genius-thinker

도메인 에이전트:
- tauri-native-dev   — Tauri 네이티브 기능 구현
- react-ui-dev       — React 데스크톱 UI 구현
- llm-integrator     — LLM API 연동 + 프롬프트 엔지니어링
- animation-dev      — 펭귄 캐릭터 애니메이션

패턴: Fan-out (UI + Native + LLM 병렬) → 통합 테스트
```
