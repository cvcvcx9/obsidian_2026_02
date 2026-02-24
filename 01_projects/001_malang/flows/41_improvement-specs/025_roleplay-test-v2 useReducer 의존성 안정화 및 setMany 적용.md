---
유형: 개선-프론트엔드
프로젝트: 001_malang
기능: roleplay-test-v2-reducer-deps-stabilization
상태: 완료
수정일: 2026-02-20
태그: [flow, improvement, admin, realtime, refactor]
---

# roleplay-test-v2 useReducer 의존성 안정화 및 setMany 적용

## 배경
- `useReducer` 전환 이후 setter alias를 콜백에서 사용하면서 `react-hooks/exhaustive-deps` 경고가 대량으로 발생했다.
- 초기화 코드가 다수의 `setXxx` 호출로 길어져 가독성과 유지보수성이 떨어졌다.

## 적용 내용
- `useCallback`/`useEffect`/`useMemo`의 dependency 배열을 실제 참조 setter 기준으로 정리해 훅 경고를 제거했다.
- `resetUiStateBeforeIssueToken`를 다중 setter 호출에서 `setManyFields` 단일 호출로 변경했다.
- `sectionActions.resetCollectedData`도 `setManyFields`로 전환해 중복 갱신 코드를 축소했다.
- `appendLog`, `setSlotFailure`, `handleResponseDone*`, `sendManualText`, `issueToken` 등 핵심 콜백의 의존성 누락을 보완했다.

## 효과
- `page.tsx` 단일 파일 기준 ESLint 훅 의존성 경고가 0건으로 정리되었다.
- UI 초기화/수집데이터 초기화가 partial state 업데이트 1회로 단순화되어 읽기 부담이 줄었다.
- reducer 전환 중간 상태에서 발생한 불안정 의존성 이슈를 먼저 정리해 다음 리팩터링 단계 진행 기반을 확보했다.

## 검증
- `yarn eslint "app/(dashboard)/roleplay-test-v2/page.tsx"`
- `yarn build`

## 변경 파일
- `admin/app/(dashboard)/roleplay-test-v2/page.tsx`
