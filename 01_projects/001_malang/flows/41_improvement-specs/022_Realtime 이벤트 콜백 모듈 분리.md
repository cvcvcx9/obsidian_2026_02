---
유형: 개선-프론트엔드
프로젝트: 001_malang
기능: realtime-callback-module-split
상태: 완료
수정일: 2026-02-20
태그: [flow, improvement, admin, realtime, refactor]
---

# Realtime 이벤트 콜백 모듈 분리

## 배경
- `page.tsx` 내 Realtime 이벤트 콜백 정의가 계속 커져 엔트리 흐름을 읽기 어려웠다.
- 커스텀 훅 대신(프로젝트 규칙), 팩토리 함수 모듈로 콜백 맵을 외부화해 가독성을 높이기로 했다.

## 적용 내용
- 신규 모듈 추가: `admin/components/roleplay-test-v2/realtime-event-handlers.ts`
  - `createRoutedRealtimeEventHandlers(deps)`
  - `createFallbackRealtimeEventHandlers(deps)`
  - deps 인터페이스를 `Routed...` / `Fallback...`로 분리
- `page.tsx`에서는 핸들러 생성(useMemo) + 조회/실행만 담당하도록 정리
- 엔트리 파일/분리 모듈 모두 각 코드 블록 상단에 역할 주석을 유지

## 효과
- `handleRealtimeMessage`와 `routeRealtimeEvent`가 "흐름 중심"으로 단순화됨
- 분석 에이전트가 엔트리 파일에서 전체 제어 흐름을 빠르게 파악 가능
- 이벤트 세부 구현은 별도 파일에서 집중적으로 수정 가능

## 검증
- `npx eslint "app/(dashboard)/roleplay-test-v2/page.tsx" "components/roleplay-test-v2/realtime-event-handlers.ts"`
- `yarn build` 통과

## 변경 파일
- `admin/app/(dashboard)/roleplay-test-v2/page.tsx`
- `admin/components/roleplay-test-v2/realtime-event-handlers.ts`
