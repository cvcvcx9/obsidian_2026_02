---
유형: 개선-프론트엔드
프로젝트: 001_malang
기능: handle-realtime-message-pipeline-split
상태: 완료
수정일: 2026-02-20
태그: [flow, improvement, admin, realtime, refactor]
---

# handleRealtimeMessage 파이프라인 4단계 분리

## 배경
- `handleRealtimeMessage`는 이벤트별 처리 분리 이후에도 공통 파이프라인 로직(파싱/중복가드/로깅/디스패치)이 본문에 밀집되어 있었다.
- 엔트리 함수는 "흐름"만 보여주고, 단계별 세부는 별도 함수로 분리하는 방향이 가독성과 유지보수에 유리하다.

## 적용 내용
- 엔트리 함수 흐름을 아래 4단계로 고정했다.
  1. `parseIncomingRealtimeEvent(raw)`
  2. `shouldSkipDuplicateRealtimeEvent(eventType, payload)`
  3. `appendRealtimeVerboseEventLog(eventType, raw)`
  4. `dispatchRealtimeFallbackEvent(eventType, payload)`
- fallback 이벤트 맵을 `realtimeFallbackEventHandlers`(useMemo)로 외부화했다.
- 각 단계 함수 상단에 역할 주석을 추가했다.

## 엔트리 파일 가독성 규칙
- `page.tsx` 상단 기능 맵 유지
- 이벤트 엔트리(`handleRealtimeMessage`)는 "파이프라인 순서"만 담당
- 이벤트 세부 동작은 핸들러 콜백으로 위임

## 기대 효과
- 엔트리 함수에서 분기 디테일 제거
- 신규 이벤트 추가 시 `realtimeFallbackEventHandlers` 중심 수정
- 리뷰 시 공통 파이프라인과 비즈니스 처리 구간을 빠르게 분리 가능

## 검증
- `npx eslint "app/(dashboard)/roleplay-test-v2/page.tsx"`

## 변경 파일
- `admin/app/(dashboard)/roleplay-test-v2/page.tsx`
