---
유형: 개선-프론트엔드
프로젝트: 001_malang
기능: route-realtime-event-map-externalization
상태: 완료
수정일: 2026-02-20
태그: [flow, improvement, admin, realtime, refactor]
---

# routeRealtimeEvent 핸들러 맵 외부화

## 배경
- `routeRealtimeEvent` 내부에서 `eventRouter`를 매 호출마다 선언하고 있어 읽기 흐름이 길었다.
- 엔트리 함수에서 "핸들러 조회/실행"만 보이도록 분리하면 분석 에이전트/리뷰어의 추적 비용이 줄어든다.

## 적용 내용
- `routeRealtimeEvent` 우선 처리 핸들러 맵을 `routedRealtimeEventHandlers`(useMemo)로 외부화했다.
- `routeRealtimeEvent`는 핸들러 조회 -> 실행 -> 처리 여부 반환만 담당하도록 축소했다.
- 신규/기존 분리 함수 상단에 역할 주석을 유지했다.

## 검증
- `npx eslint "app/(dashboard)/roleplay-test-v2/page.tsx"`

## 변경 파일
- `admin/app/(dashboard)/roleplay-test-v2/page.tsx`
