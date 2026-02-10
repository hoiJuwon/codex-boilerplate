---
name: debug
description: Root-cause-first debugging workflow for bug fixes and unexpected behavior. Use when symptoms are unclear or reproducibility is unstable. Force diagnosis evidence before code edits, then apply the smallest fix with regression verification.
---

# Debug

증상이 아니라 원인을 고친다. 진단 없이 수정하지 않는다.

## 실행 원칙

- 진단 전 코드 수정 금지
- 분할정복으로 문제 범위를 좁힌다
- 관찰 가능한 증거(로그, 조건, 코드 경로)로 원인을 확정한다
- 임시 우회보다 재발 방지 수정을 우선한다

## Phase 1: 진단 (수정 금지)

1. 증상을 명확히 정의한다.
   - 예상 동작
   - 실제 동작
   - 재현 조건
2. 재현 커맨드를 만든다.
   - 테스트 커맨드 또는 최소 재현 CLI 시나리오
3. 분할정복으로 범위를 좁힌다.
   - "어디까지 정상인가?"를 연속 확인
   - 진입점 → 중간 계층 → 분기점 순서로 절반씩 축소
4. 분기와 데이터 흐름을 확인한다.
   - 조건문, early return, 예외 처리, 변환 지점
5. "왜?"를 반복해 근본 원인을 문장으로 고정한다.

## Phase 2: 진단 보고

아래 형식으로 요약한다.

```text
[진단 결과]
증상:
예상 동작:
실제 동작:

실행 흐름:
1. ... (정상)
2. ... (정상)
3. ... (비정상)

근본 원인:
증거:

제안 수정:
- 변경 포인트
- 영향 범위
```

진단이 끝나면 사용자에게 방향 확인을 받는다. 사용자가 즉시 수정을 요청한 경우에는 바로 Phase 3로 진행해도 된다.

## Phase 3: 수정 + 검증

1. 가장 작은 수정으로 원인을 제거한다.
2. 재현 테스트를 추가/수정해 회귀를 막는다.
3. 수정 전 실패, 수정 후 통과를 확인한다.
4. 관련 린트/타입체크/영향 테스트를 실행한다.

## 금지 사항

- 원인 미확정 상태에서 "일단 되는" 패치
- 하드코딩 우회
- 에러를 삼켜서 증상만 숨기는 처리
- 검증 없이 추측으로 종료
