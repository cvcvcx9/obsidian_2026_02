---
유형: 개선-프론트엔드
프로젝트: 001_malang
기능: issue-token-reset-split
상태: 완료
수정일: 2026-02-20
태그: [flow, improvement, admin, refactor, readability]
---

# issueToken 초기화 블록 분리

## 배경
- `issueToken` 내부에 UI 상태 reset + ref 캐시 초기화가 길게 섞여 있어 본래 목적(세션 발급 API 호출)을 읽기 어려웠다.
- 리팩터링 시 회귀 위험을 낮추기 위해 "초기화 책임"을 함수 단위로 분리할 필요가 있었다.

## 적용 내용
- `resetRealtimeRefsBeforeIssueToken`
  - Realtime 관련 ref/맵 캐시 초기화 담당
- `resetUiStateBeforeIssueToken`
  - 화면 상태(useState) 초기화 담당
- `issueToken`에서는 위 두 함수를 호출한 뒤 API 호출/동기화 흐름만 유지

## 기대 효과
- `issueToken` 본문 가독성 개선
- 초기화 범위 변경 시 수정 포인트 명확화
- 향후 "세션 리셋" 공통화 작업의 기반 확보

## 검증
- `npx eslint "app/(dashboard)/roleplay-test-v2/page.tsx" "components/roleplay-test-v2/realtime-event-handlers.ts"`

## 변경 파일
- `admin/app/(dashboard)/roleplay-test-v2/page.tsx`
