---
유형: 개선-프론트엔드
프로젝트: 001_malang
기능: roleplay-test-v2-shared-module-split
상태: 완료
수정일: 2026-02-20
태그: [flow, improvement, admin, realtime, refactor]
---

# roleplay-test-v2 공통 유틸 및 시나리오 맵 분리

## 배경
- `page.tsx` 상단에 이벤트 키 유틸/로그 유틸/시나리오 stage 맵이 함께 존재해 파일 길이가 길고 진입 가독성이 떨어졌다.
- 이벤트 콜백 모듈 분리 이후, 공통 유틸도 모듈 단위로 이동해 엔트리 파일을 더 슬림화할 필요가 있었다.

## 적용 내용
- 신규 모듈 추가
  - `admin/components/roleplay-test-v2/realtime-event-utils.ts`
    - `buildRealtimeEventKey`
    - `buildActionExecutionKey`
    - `getActionPayloadRecord`
    - `formatLogFields`
    - `parseRealtimeEvent`
    - `sanitizeEventForLog`
    - `stableSerialize`
  - `admin/components/roleplay-test-v2/realtime-scenario.ts`
    - `ScenarioStage` 타입
    - 시나리오 stage 맵
    - `getScenarioStageMap`
    - `getPendingStages`
    - `buildSlotDetectionEvent`
- `page.tsx`에서는 위 모듈 import 후 엔트리 제어 흐름 중심으로 유지

## 효과
- 엔트리 파일에서 "공통 유틸 구현"이 제거되어 핵심 흐름 가독성 향상
- 이벤트 키/로그/시나리오 관련 재사용 포인트 확보
- 향후 테스트 작성 시 유틸 모듈 단위 검증 가능

## 검증
- `npx eslint "app/(dashboard)/roleplay-test-v2/page.tsx" "components/roleplay-test-v2/realtime-event-handlers.ts" "components/roleplay-test-v2/realtime-event-utils.ts" "components/roleplay-test-v2/realtime-scenario.ts"`
- `yarn build` 통과

## 변경 파일
- `admin/app/(dashboard)/roleplay-test-v2/page.tsx`
- `admin/components/roleplay-test-v2/realtime-event-utils.ts`
- `admin/components/roleplay-test-v2/realtime-scenario.ts`
