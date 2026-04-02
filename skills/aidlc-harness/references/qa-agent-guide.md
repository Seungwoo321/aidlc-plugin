<!--
  Copyright (c) revfactory (https://github.com/revfactory/harness)
  Licensed under the Apache License, Version 2.0
  https://www.apache.org/licenses/LICENSE-2.0

  Modified by Seungwoo Lee for AIDLC Plugin.
  Changes: AIDLC 방법론 맥락에 맞게 수정, 요약, AIDLC 프로젝트 예시 추가
-->

# QA 에이전트 설계 가이드

> 출처: revfactory/harness (Apache-2.0)

하네스에서 QA/검증 에이전트를 생성할 때 참고하는 가이드. 실제 프로젝트에서 발견된 버그 패턴과 근본 원인 분석을 바탕으로, QA가 놓치기 쉬운 결함을 체계적으로 잡는 검증 방법론을 제공한다.

---

## 1. QA 에이전트가 놓치는 결함의 패턴

### 경계면 불일치 (Boundary Mismatch)

가장 빈번한 결함. 두 컴포넌트가 각각 "올바르게" 구현되어 있지만, 연결 지점에서 계약이 어긋남.

| 경계면 | 불일치 예시 |
|--------|-----------|
| API 응답 → 프론트 훅 | API가 `{ projects: [...] }` 반환, 훅이 배열 기대 |
| API 응답 필드명 → 타입 정의 | `thumbnailUrl`(camelCase) vs `thumbnail_url`(snake_case) |
| 파일 경로 → 링크 href | 페이지가 `/dashboard/create`에 있는데 링크가 `/create` |
| 상태 전이 맵 → 실제 status 업데이트 | 맵에 전이 정의, 코드에서 전환 누락 |
| 즉시 응답 → 비동기 결과 | API가 즉시 `{ status }` 반환, 프론트가 `data.failedIndices` 접근 |

### 왜 정적 코드 리뷰로 못 잡나

- TypeScript 제네릭의 한계: `fetchJson<T>()` — 런타임 응답과 T가 달라도 컴파일 통과
- `npm run build` 통과 ≠ 정상 동작
- 존재 검증 vs 연결 검증의 차이

---

## 2. 통합 정합성 검증 (Integration Coherence Verification)

### API 응답 ↔ 프론트 훅 타입 교차 검증

```
1. API route에서 NextResponse.json()에 전달하는 객체의 shape 추출
2. 대응 훅에서 fetchJson<T>의 T 타입 확인
3. shape과 T가 일치하는지 비교
4. 래핑 여부 확인 (API가 { data: [...] }를 반환하면 훅이 .data를 꺼내는지)
```

### 파일 경로 ↔ 링크/라우터 경로 매핑

```
1. src/app/ 하위 page.tsx 파일 경로에서 URL 패턴 추출
2. 코드 내 모든 href=, router.push(, redirect( 값 수집
3. 각 링크가 실제 존재하는 page 경로와 매칭되는지 확인
```

### 상태 전이 완전성 추적

```
1. STATE_TRANSITIONS에서 허용된 전이 목록 추출
2. 모든 API route에서 .update({ status: "..." }) 패턴 검색
3. 각 전이가 맵에 정의되어 있는지 확인
4. 맵에 정의된 전이 중 코드에서 실행되지 않는 것 식별 (죽은 전이)
```

---

## 3. QA 에이전트 설계 원칙

### general-purpose 타입 사용

QA 에이전트가 `Explore` 타입이면 읽기만 가능. 효과적인 QA는 Grep, 스크립트 실행, 필요 시 수정까지 필요.

### "교차 비교" 우선

| 약한 체크리스트 | 강한 체크리스트 |
|---------------|---------------|
| API 엔드포인트가 존재하는가? | API 응답 shape과 훅 타입이 일치하는가? |
| 상태 전이 맵이 정의되어 있는가? | 모든 status 업데이트가 맵과 일치하는가? |
| 페이지 파일이 존재하는가? | 모든 링크가 실제 페이지를 가리키는가? |

### "양쪽을 동시에 읽어라" 원칙

경계면 버그를 잡으려면 한쪽만 읽어선 안 된다. 반드시:
- API route **와** 대응 훅을 **같이** 읽고
- 상태 전이 맵 **와** 실제 업데이트 코드를 **같이** 읽어야 한다

### Incremental QA

QA는 전체 완성 후가 아니라, 각 모듈 완성 직후에 실행한다.

---

## 4. 검증 체크리스트 템플릿

### 통합 정합성 검증 (웹 앱)

#### API ↔ 프론트엔드 연결
- [ ] 모든 API route의 응답 shape과 대응 훅의 제네릭 타입이 일치
- [ ] 래핑된 응답은 훅에서 unwrap하는지 확인
- [ ] snake_case ↔ camelCase 변환이 일관되게 적용
- [ ] 모든 API 엔드포인트에 대응하는 프론트 훅이 존재

#### 라우팅 정합성
- [ ] 코드 내 모든 href/router.push 값이 실제 page 파일 경로와 매칭
- [ ] route group이 URL에서 제거되는 것을 고려한 경로 검증

#### 상태 머신 정합성
- [ ] 정의된 모든 상태 전이가 코드에서 실행됨
- [ ] 코드의 모든 status 업데이트가 전이 맵에 정의됨
- [ ] 프론트에서 상태 기반 분기의 상태값이 실제 도달 가능

#### 데이터 흐름 정합성
- [ ] DB 스키마 필드명과 API 응답 필드명 매핑 일관
- [ ] 옵셔널 필드에 대한 null/undefined 처리 양쪽 일관
