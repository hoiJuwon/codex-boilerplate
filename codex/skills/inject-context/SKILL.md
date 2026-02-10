---
name: inject-context
description: Context injection workflow that identifies target project/domain from a task, loads only required docs and architecture files, summarizes context, and then executes the requested work. Use when requests are broad, cross-module, or require architecture-aware implementation.
---

# Inject Context

작업을 시작하기 전에 필요한 컨텍스트를 자동으로 좁혀서 주입하고, 바로 실행까지 이어간다.

## 실행 원칙

- 필요한 컨텍스트만 로드한다.
- 긴 문서는 키워드 기반으로 필요한 섹션만 추출한다.
- 컨텍스트 요약 후 멈추지 말고 실제 작업까지 수행한다.
- 동일 문서를 중복 로드하지 않는다.

## 1단계: 프로젝트/도메인 판별

입력(요청 본문 + 파일 경로)에서 프로젝트와 도메인을 판별한다.

### 프로젝트 키워드

| 키워드 | 프로젝트 |
| --- | --- |
| `server`, `api`, `backend`, `서버`, `백엔드`, `모듈`, `서비스` | `server` |
| `web`, `frontend`, `ui`, `프론트`, `컴포넌트`, `페이지` | `web` |
| `mobile`, `flutter`, `앱` | `mobile` |
| `package`, `packages`, `library`, `sdk`, `runtime`, `모듈스펙` | `packages` |
| `infra`, `devops`, `docker`, `k8s`, `배포` | `infra` |

### 파일 경로 힌트

| 경로 패턴 | 프로젝트 |
| --- | --- |
| `server/src/*` | `server` |
| `web/src/*` | `web` |
| `mobile/lib/*` | `mobile` |
| `packages/*` | `packages` |
| `apps/*` | `apps` |

### 도메인 키워드 (문서 선택 힌트)

도메인 키워드가 있으면, 아래 우선순위로 문서를 찾는다.

1. `docs/tech/` (기술 계약, 규약, 아키텍처)
2. `docs/` (온보딩, 운영, 설계)
3. `prd/` (태스크 단위 요구사항)
4. `README.md` / `ARCHITECTURE.md` / `CONTRIBUTING.md`

찾는 방법:

- `rg -n "<키워드>" docs docs/tech prd README.md ARCHITECTURE.md` (존재하는 경로만)
- 문서가 길면 필요한 섹션만 `sed -n`으로 발췌한다.

## 2단계: 코어 컨텍스트 로드

프로젝트별 핵심 규칙 문서를 먼저 읽는다.

- `AGENTS.md` (존재 시)
- `.claude/CLAUDE.md` 또는 `CLAUDE.md` (존재 시)
- `docs/tech/` 관련 문서 (존재 시)
- 요청이 문서 기반이면 대상 task/spec/PRD 문서
- 모노레포면 `<project>/AGENTS.md`, `<project>/README.md` (존재 시)

## 3단계: 도메인 문서 선택 로드

1. 1단계에서 잡은 도메인 문서만 로드한다.
2. 문서가 길면 전체 로드하지 않는다.
   - 먼저 `rg -n "<키워드>" <문서>`로 관련 섹션 위치를 찾는다.
   - 필요한 구간만 `sed -n`으로 발췌한다.
3. 500줄+ 문서는 요약 발췌 원칙을 고수한다.

## 4단계: 코드 아키텍처 탐색

프로젝트별 우선 탐색 경로:

| 프로젝트 | 탐색 경로 |
| --- | --- |
| `server` | `server/src/`, `server/test/` |
| `web` | `web/src/`, `web/test/` |
| `mobile` | `mobile/lib/`, `mobile/test/` |
| `packages` | `packages/*/src/`, `packages/*/test/` |
| `apps` | `apps/*/src/`, `apps/*/test/` |

탐색 시 반드시 포함:

- 관련 모듈/컴포넌트 위치
- 핵심 함수/타입
- 데이터 흐름
- 유사 구현 패턴
- 기존 테스트 위치

경로 미존재 fallback:

1. 탐색 경로가 없으면 `find <project-root> -maxdepth 3 -type d`로 실제 후보를 찾는다.
2. 후보 중 `src`, `modules`, `services`, `test`를 우선 선택한다.
3. fallback 경로를 요약에 명시한다.

## 5단계: 컨텍스트 요약

작업 전에 아래 4개를 짧게 정리한다.

- 판별된 프로젝트/도메인
- 로드한 문서와 핵심 규칙
- 관련 코드 진입점
- 바로 수행할 실행 계획

## 6단계: 작업 수행

요청 성격에 따라 바로 실행한다.

- 조사 요청: 근거 파일과 함께 답변
- 수정 요청: 구현 + 검증까지 수행
- 설계 요청: 선택지와 트레이드오프를 제시하고 확정 후 진행

## 체크리스트

- 컨텍스트 로드에서 끝나지 않았는가
- 불필요한 대형 문서를 통째로 읽지 않았는가
- 실제 코드/테스트 근거를 포함했는가
