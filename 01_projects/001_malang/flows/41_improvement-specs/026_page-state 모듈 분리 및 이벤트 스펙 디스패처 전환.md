---
유형: 개선-프론트엔드
프로젝트: 001_malang
기능: roleplay-test-v2-container-spec-reducer-step
상태: 완료
수정일: 2026-02-20
태그: [flow, improvement, admin, realtime, refactor]
---

# page-state 모듈 분리 및 이벤트 스펙 디스패처 전환

## 배경
- `roleplay-test-v2/page.tsx`에 상태 타입/리듀서/초기값/리셋 로직과 이벤트 라우팅 로직이 함께 있어 컨테이너 집중도가 높았다.
- `response.canceled/cancelled/failed`처럼 형태가 같은 이벤트 핸들러가 반복되어 유지보수 비용이 증가했다.

## 적용 내용
- 신규 상태 모듈 추가: `admin/components/roleplay-test-v2/page-state.ts`
  - `PageState`, `SessionRuntime`, `StateUpdater`
  - `createInitialPageState`, `pageStateReducer`
  - `createPageStateSetters` (setField 기반 setter 팩토리)
  - `defaultScenarios`, `DEFAULT_VAD_THRESHOLD`, `DEFAULT_SILENCE_DURATION_MS`
  - reducer action 확장: `resetForIssueToken`, `resetCollectedData`
- `page.tsx`에서 state 정의/초기화 구현을 제거하고 모듈 import로 전환했다.
- `page.tsx`의 긴 setter alias 객체를 제거하고 `createPageStateSetters(setField)` 호출로 대체했다.
- `issueToken` 사전 UI 초기화를 `dispatchPageState({ type: "resetForIssueToken" })` 1줄로 전환했다.
- `sectionActions.resetCollectedData`를 `dispatchPageState({ type: "resetCollectedData" })`로 전환했다.
- `realtime-event-handlers.ts`를 선언형 스펙 기반으로 재구성했다.
  - `RoutedEventSpec` + `logPolicy`(`guard`/`verbose`) 도입
  - 공통 executor(`createRoutedEventHandler`)가 guard/log/handler를 순서대로 처리
  - `response.canceled/cancelled/failed`는 `createResponseTerminalEventSpec` 템플릿으로 통합

## 효과
- `page.tsx`에서 상태 보일러플레이트가 분리되어 컨테이너 역할(와이어링/오케스트레이션)에 더 집중된다.
- 다건 리셋이 action 명칭 기반으로 표준화되어 의도가 코드에서 바로 드러난다.
- 반복 이벤트 패턴이 템플릿화되어 신규 terminal 이벤트 추가 시 확장 비용이 낮아졌다.

## 검증
- `yarn eslint "app/(dashboard)/roleplay-test-v2/page.tsx" "components/roleplay-test-v2/page-state.ts" "components/roleplay-test-v2/realtime-event-handlers.ts"`
- `yarn build`

## 변경 파일
- `admin/app/(dashboard)/roleplay-test-v2/page.tsx`
- `admin/components/roleplay-test-v2/page-state.ts`
- `admin/components/roleplay-test-v2/realtime-event-handlers.ts`
