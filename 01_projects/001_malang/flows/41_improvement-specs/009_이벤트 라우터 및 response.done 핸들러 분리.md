---
유형: 개선-구현
프로젝트: 001_malang
기능: realtime-event-router-refactor
상태: 완료
수정일: 2026-02-20
태그: [flow, improvement, realtime, router, handler]
---

# 이벤트 라우터 및 response.done 핸들러 분리

## 왜 이 작업을 했는가
- `handleRealtimeMessage`에 이벤트 분기와 도메인 로직이 섞여 있어, 이벤트 추가/수정 시 회귀 위험이 컸다.
- 특히 `response.done`은 목적별(`slot_detection`, `text_enrichment`, `fsm_reply`) 로직이 한 블록에 모여 복잡도가 높았다.

## 이번에 적용한 내용

### 1) 라우팅 계층 추가
- `routeRealtimeEvent(eventType, payload)`를 추가해 이벤트 라우팅을 테이블 기반으로 분리했다.
- `eventRouter`에서 아래 이벤트를 처리한다.
  - `session.updated`
  - `response.created`
  - `conversation.item.created`
  - `response.canceled`, `response.cancelled`, `response.failed`
  - `conversation.item.deleted`
  - `response.done`

### 2) response.done 목적별 핸들러 분리
- `handleResponseDoneEvent`를 중심으로 목적별 핸들러를 분리했다.
  - `handleResponseDoneSlotDetection`
  - `handleResponseDoneTextEnrichment`
  - `handleResponseDoneFsmReply`
- 기존 거대 분기에서 목적별 처리 책임을 분리해 가독성과 유지보수성을 개선했다.

### 3) 기존 흐름 유지
- 이벤트 정규화(`parseRealtimeEvent`) + dedupe + stale mapping GC는 기존대로 선행 처리한다.
- 라우터에서 처리되지 않은 이벤트만 `handleRealtimeMessage`의 후속 분기(음성 전사/오디오 transcript)로 내려간다.

## 기대 효과
- 이벤트별 책임 경계가 명확해져 수정 영향 범위가 줄어든다.
- `response.done` 관련 회귀 포인트가 줄고, 목적별 디버깅이 쉬워진다.
- 다음 단계(전사 가드 공통화, action 유틸화)로 분해 작업을 이어가기 쉬워졌다.

## 검증
- ESLint 확인: `yarn eslint "app/(dashboard)/roleplay-test-v2/page.tsx"` 통과
- 참고: 전체 `yarn build` 실패는 기존 타입 이슈(다른 훅 파일)로, 이번 변경 파일과 별개

## 변경 파일
- `admin/app/(dashboard)/roleplay-test-v2/page.tsx`

## 연결 노트
- [[01_projects/001_malang/flows/41_improvement-specs/006_개선 실행 체크리스트|개선 실행 체크리스트]]
- [[01_projects/001_malang/flows/41_improvement-specs/007_idempotency 적용 및 맵 정리 공통화|idempotency 적용 및 맵 정리 공통화]]
