---
name: loop
description: Close-the-loop implementation workflow. Use for feature work, bug fixes, and refactors that must be completed with executable CLI verification evidence (tests/lint/build, and E2E where relevant) before stopping. Enforce test-first (RED-GREEN), pattern matching with existing code, and real local infrastructure over mocks.
---

# Loop

구현과 검증을 하나의 루프로 끝낸다. "코드를 바꾸는 것"이 아니라 "동작을 증명하는 것"이 완료 조건이다.

## 실행 원칙

- 검증 불가능한 코드는 작성하지 않는다.
- 가능하면 테스트 먼저 작성하고 RED를 확인한 뒤 구현한다.
- 기존 패턴을 먼저 찾고 그 패턴을 따른다.
- mock/stub/fake보다 실제 로컬 인프라 기반 검증을 우선한다.
- CLI 실행 결과가 없으면 루프를 닫지 않는다.
- 독립적인 탐색/검증 작업은 병렬로 실행해 시간을 줄인다.

## Phase 0: 인프라/의존성 확인

1. 로컬 인프라가 필요한 작업인지 확인한다.
   - DB/Redis/Queue 등 외부 의존성이 있거나 E2E/통합 테스트가 필요하면 "필수"로 간주한다.
2. Docker를 쓰는 레포면 compose 파일을 찾고 상태를 확인한다.
   - 후보 찾기: `find . -maxdepth 4 -type f \( -name 'docker-compose*.yml' -o -name 'compose*.yml' \)`
   - 상태 확인 예시: `docker compose -f <compose.yml> ps`
3. 레포에 인프라 시작 스크립트가 있으면 그걸 우선한다.
   - 예: `./scripts/*start*.sh`, `./scripts/dev-up.sh`
4. 인프라가 내려가 있거나 필수 ENV/권한이 없으면 즉시 안내하고 멈춘다.

## Phase 1: 스코프와 코드 지형 파악 (수정 금지)

1. 입력 형태를 분류한다.
   - 파일 경로: 파일 요구사항 추출
   - 브랜치/커밋/PR 맥락: `git diff` 기반 변경 범위 추출
   - 자유 텍스트: 대상 프로젝트/모듈 추론
2. 대상 프로젝트(또는 패키지/앱)를 확정한다.
   - 예: `server`, `web`, `mobile`, `packages`, `apps/*`
3. 관련 파일을 빠르게 수집한다.
   - `rg --files`, `rg -n`으로 구현/테스트/설정 파일을 함께 찾는다.
4. 기존 구현 패턴을 확인한다.
   - 유사 기능 파일 2~3개를 찾고 구조, 네이밍, 에러 처리 패턴을 기록한다.

## Phase 2: Verification Audit

코드 경로를 감으로 판단하지 말고 목록화한다.

1. 코드 경로를 식별한다.
   - 정상 흐름
   - 검증/가드 실패 흐름
   - 예외/에러 흐름
   - 외부 연동(DB/API/이벤트) 흐름
2. 각 경로의 테스트 현황을 기록한다.
   - 기존 테스트 위치
   - 커버리지 충분/부족
   - 현재 요구사항과 기대값 정합성
3. 액션을 분류한다.
   - 유지
   - 수정(기획 불일치)
   - 보강(커버리지 부족)
   - 신규(테스트 없음)

## Phase 3: 검증 구축 (테스트 우선)

1. 테스트를 먼저 정비한다.
   - 기존 테스트에 케이스 추가를 우선
   - 필요한 경우에만 새 테스트 파일 생성
2. RED를 확인한다.
   - 기존 테스트가 먼저 통과하는지 확인
   - 새/수정 테스트가 구현 전 실패하는지 확인
3. 표준 테스트로 검증이 어려우면 ad-hoc CLI 검증 스크립트를 만든다.
   - 검증 목적이 끝나면 정리 가능

## Phase 4: 구현

1. 변경 항목을 작은 단위로 나눈다.
2. 각 변경마다 근거를 연결한다.
   - 어떤 테스트를 GREEN으로 만들기 위한 변경인지 명확히 유지
3. 기존 코드 스타일/아키텍처 경계를 유지한다.
4. 임시 우회(TODO, placeholder, 하드코딩)로 종료하지 않는다.

## Phase 5: 검증 루프

프로젝트별 검증 명령을 실행한다. 실행 위치를 명확히 고정한다.

1. 해당 프로젝트의 "표준 검증 커맨드"를 먼저 확인한다.
   - `package.json` scripts, `Makefile`, `README.md`, `docs/tech/*`, CI workflow
2. 가능한 한 "빠른 것"부터 실행한다.
   - format/lint → typecheck → unit/integration tests → build → e2e/smoke (존재 시)
3. E2E가 있으면, 변경 범위를 커버하는 최소 세트를 먼저 실행한다.
   - 매핑이 불명확하면 smoke 또는 대표 시나리오 1개 → 필요 시 전체 E2E
4. 실행한 커맨드와 결과를 기록한다. (Phase 6 종료 보고에 포함)

## Phase 6: 루프 종료 보고

아래를 요약해서 종료한다.

- 변경 파일과 핵심 의도
- 추가/수정된 테스트와 커버 경로
- 실제 실행한 검증 명령과 결과
- 남은 리스크(있다면)와 다음 액션

## 종료 금지 조건

다음 중 하나라도 해당되면 종료하지 않는다.

- RED/GREEN 흐름이 확인되지 않음
- 실제 검증 명령 실행 결과가 없음
- 변경 코드 경로 중 미검증 구간이 남음
- mock 기반 검증만 있고 실제 경로 검증이 없음
- 서버 변경인데 인프라/타깃 E2E 검증이 누락됨
