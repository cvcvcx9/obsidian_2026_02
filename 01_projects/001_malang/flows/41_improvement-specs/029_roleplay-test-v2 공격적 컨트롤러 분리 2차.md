---
유형: 개선-프론트엔드
프로젝트: 001_malang
기능: roleplay-test-v2-aggressive-controller-split-phase2
상태: 완료
수정일: 2026-02-20
태그: [flow, improvement, admin, realtime, refactor]
---

# roleplay-test-v2 공격적 컨트롤러 분리 2차

## 배경
- 1차 분리 이후에도 `page.tsx` 내부에 전사/슬롯 감지/텍스트 보강 helper가 중복으로 남아 있었다.
- `controllers/realtime-turn-utils.ts`를 이미 import 중이었지만, 페이지 내부 동일 함수 선언이 공존해 유지보수 포인트가 이중화돼 있었다.
- `controllers/realtime-fsm-controller.ts`는 생성되어 있었으나 실제 wiring이 없어 dead module 상태였다.

## 적용 내용
- `admin/app/(dashboard)/roleplay-test-v2/page.tsx`
  - 아래 중복 helper를 페이지에서 제거하고 `realtime-turn-utils` 구현만 사용하도록 정리
    - `extractAssistantText`
    - `decideTranscriptGuard`
    - `getPendingStages`
    - `buildSlotDetectionEvent`
    - `buildTextEnrichmentEvent`
    - `extractResponseDoneText`
    - `parseSlotDetectionResult`
    - `filterExtractedSlotsByPending`
    - `parseStaffTextEnrichment`
  - 정리 후 미사용 import 제거(`getScenarioStageMap`, `ScenarioStage`, `StaffUtteranceEnrichment`, `TranscriptGuardDecision`)
- `admin/components/roleplay-test-v2/controllers/realtime-fsm-controller.ts`
  - 미연결 상태의 dead module 제거

## 효과
- `roleplay-test-v2/page.tsx` 라인 수가 `2578 -> 2152`로 감소(약 426라인 축소)
- 1차 대비 누적 축소: `3211 -> 2152`(약 1059라인 축소)
- 이벤트 턴 관련 파싱/가드 정책이 `realtime-turn-utils` 단일 소스로 수렴되어 변경 안정성 개선

## 검증
- LSP 진단
  - `admin/app/(dashboard)/roleplay-test-v2/page.tsx`: diagnostics 없음
  - `admin/components/roleplay-test-v2/controllers/realtime-turn-utils.ts`: diagnostics 없음
- ESLint(변경 범위)
  - `yarn eslint "app/(dashboard)/roleplay-test-v2/page.tsx" "components/roleplay-test-v2/controllers/realtime-turn-utils.ts"`
- Build
  - `yarn build` 통과

## 변경 파일
- `admin/app/(dashboard)/roleplay-test-v2/page.tsx`
- `admin/components/roleplay-test-v2/controllers/realtime-turn-utils.ts`
- `admin/components/roleplay-test-v2/controllers/realtime-fsm-controller.ts`(삭제)
