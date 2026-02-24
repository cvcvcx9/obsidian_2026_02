---
유형: 개선-구현
프로젝트: 001_malang
기능: action-handler-commonization-and-log-standardization
상태: 완료
수정일: 2026-02-20
태그: [flow, improvement, action, log, refactor]
---

# action 처리 공통 유틸화 및 로그 키 표준화

## 왜 필요한가
- `applyFsmActions` 내부의 action 분기가 길어지면서 payload 파싱/처리 패턴이 중복되었다.
- guard 로그 포맷이 문자열 템플릿 중심이라 검색 키가 일관되지 않았다.

## 어떤 작업을 했는가

### 1) action 처리 공통 유틸화
- `getActionPayloadRecord(action)` 유틸을 추가해 payload object 파싱 중복을 줄였다.
- `applyFsmActions`에 `actionHandlers` 테이블을 도입해 액션 타입별 처리를 함수로 분리했다.
- 루프는 dedupe 확인 후 `actionHandlers[action.type]` 호출 구조로 단순화했다.

### 2) 로그 포맷/키 표준화
- `formatLogFields(fields)` 유틸을 추가해 key=value 로그 조합을 공통화했다.
- `appendGuardLog(key, fields)`를 추가해 guard 로그를 `[GUARD:<key>]` 형식으로 통일했다.
- 주요 guard 로그를 새 형식으로 교체했다.
  - `response.cancel`
  - `conversation.item.delete`
  - `assistant.item.created`
  - `response.map.cleanup`
  - `item.map.cleanup`
  - `stale-mapping-gc`
  - `action.skip-duplicate`
  - `response.canceled/cancelled/failed`
  - `conversation.item.deleted`

## 기대 효과
- action 타입이 늘어나도 handler 한 개 추가로 확장 가능하다.
- 로그 필드 검색(`responseId=...`, `itemId=...`)이 쉬워져 디버깅 속도가 올라간다.
- 다음 단계 검증(중복/역순/수동 이벤트)에서 로그 추적 품질이 개선된다.

## 검증
- ESLint 확인: `yarn eslint "app/(dashboard)/roleplay-test-v2/page.tsx"` 통과

## 변경 파일
- `admin/app/(dashboard)/roleplay-test-v2/page.tsx`

## 연결 노트
- [[01_projects/001_malang/flows/41_improvement-specs/006_개선 실행 체크리스트|개선 실행 체크리스트]]
- [[01_projects/001_malang/flows/41_improvement-specs/010_전사 가드 로직 공통화|전사 가드 로직 공통화]]
