---
유형: 개선-요약
프로젝트: 001_malang
상태: 진행중
수정일: 2026-02-20
태그: [flow, improvement, summary, realtime]
---

# idempotency 적용 및 맵 정리 공통화

## 왜 필요한 작업인가
- Realtime 이벤트는 중복 수신/순서 역전이 발생할 수 있어, 동일 작업이 여러 번 실행되면 상태가 꼬일 위험이 크다.
- 기존 구조는 response/item 맵 정리가 여러 분기에 흩어져 있어, 일부 경로만 수정되면 회귀 버그가 생기기 쉬웠다.
- 따라서 우선순위를 "안정성 확보 -> 맵 정리 일관화 -> 책임 분리"로 두고 단계적으로 개선해야 했다.

## 어떤 작업을 했는가

### 1) Idempotency 적용
- `sendFsmEvent`에 `idempotencyKey`를 추가하고 중복 키 캐시로 재호출을 차단했다.
- `applyFsmActions`에 action dedupe key를 추가해 짧은 시간 중복 실행을 막았다.
- Realtime 수신 이벤트에 dedupe key를 적용해 동일 이벤트 재처리를 차단했다.
- 연결 해제/재연결 시 dedupe 캐시를 초기화하도록 반영했다.

### 2) 맵 정리 공통화 + stale GC
- 공통 함수 추가:
  - `registerResponsePurposeMapping`
  - `registerAssistantItemMapping`
  - `cleanupResponseMapping`
  - `cleanupAssistantItemMapping`
- `response.created`, `response.done`, `response.canceled/cancelled/failed`, `conversation.item.created/deleted` 경로에서 공통 함수를 사용하도록 통일했다.
- stale mapping GC를 추가해 request/response/item 매핑을 TTL 기준으로 정리한다.

### 3) 이벤트 책임 분리 시작
- `parseRealtimeEvent(raw)` 정규화 함수를 분리해 `handleRealtimeMessage` 진입부 파싱 책임을 분리했다.

## 현재까지 기대 효과
- 중복 이벤트/중복 클릭 시 부수효과가 1회로 수렴할 가능성이 커졌다.
- 맵 정리 규칙이 한 곳으로 모여 유지보수와 디버깅이 쉬워졌다.
- 3단계(라우터/핸들러 분리)로 확장하기 쉬운 구조가 됐다.

## 다음 작업
- `routeRealtimeEvent + eventRouter` 테이블 전환
- `response.done` 목적별 핸들러 분리(slot_detection / fsm_reply / text_enrichment)
- 검증 시나리오 문서화(done->canceled, canceled->done, 중복 수신)

## 연결 노트
- [[01_projects/001_malang/flows/improvements/006_개선 실행 체크리스트|개선 실행 체크리스트]]
- [[01_projects/001_malang/flows/improvements/005_맵 정리 공통화|맵 정리 공통화]]
- [[01_projects/001_malang/flows/improvements/004_리얼타임 목적-응답-아이템 맵 정리 개선안|리얼타임 목적-응답-아이템 맵 정리 개선안]]
