---
유형: 개선-프론트엔드
프로젝트: 001_malang
기능: roleplay-test-v2-aggressive-controller-split-phase1
상태: 완료
수정일: 2026-02-20
태그: [flow, improvement, admin, realtime, refactor]
---

# roleplay-test-v2 공격적 컨트롤러 분리 1차

## 배경
- `page.tsx`가 3,000+ 라인 규모로 유지되어 이벤트 처리/운영 API/섹션 뷰모델이 단일 파일에 집중돼 있었다.
- 기존 분리(`page-state`, `realtime-event-handlers`, `realtime-lifecycle-controller`) 이후에도 핵심 오케스트레이션 밀도가 높아 추가 분리가 필요했다.

## 적용 내용
- 신규 모듈 추가
  - `admin/components/roleplay-test-v2/controllers/roleplay-ops-controller.ts`
    - `issueTokenOp`
    - `loadScenarioOptionsOp`
    - `runRealtimeMorphAnalysisOp`
    - `loadRoleplayStatsOp`
    - `pollPersistStatusOp`
    - `persistConversationHistoryOp`
  - `admin/components/roleplay-test-v2/controllers/realtime-turn-controller.ts`
    - response.done/전사/assistant transcript 관련 턴 처리 로직 분리
    - `handleResponseDoneEventTurn`
    - `handleSpeechStartedRealtimeEventTurn`
    - `handleUserTranscriptCompletedRealtimeEventTurn`
    - 기타 턴 핸들러
  - `admin/components/roleplay-test-v2/controllers/section-view-model.ts`
    - `buildSectionState`
    - `buildSectionActions`
- `page.tsx`는 위 모듈 호출 중심으로 치환
  - 운영 API 호출 본문을 컨트롤러 호출로 변경
  - 턴 핸들러 본문을 `realtimeTurnControllerDeps` + 위임 구조로 변경
  - 섹션 state/actions 객체 생성 본문을 뷰모델 빌더 호출로 변경

## 효과
- `roleplay-test-v2/page.tsx` 라인 수가 `3211 -> 2816`으로 감소(약 395라인 축소)
- 대형 콜백 본문이 파일 바깥으로 이동해 읽기 흐름(와이어링/오케스트레이션 중심) 개선
- 추가 분리(2차) 시 동일 컨트롤러 경계를 재활용할 수 있는 기반 확보

## 검증
- `yarn eslint "app/(dashboard)/roleplay-test-v2/page.tsx" "components/roleplay-test-v2/controllers/realtime-turn-controller.ts" "components/roleplay-test-v2/controllers/roleplay-ops-controller.ts" "components/roleplay-test-v2/controllers/section-view-model.ts"`
- `yarn build`

## 변경 파일
- `admin/app/(dashboard)/roleplay-test-v2/page.tsx`
- `admin/components/roleplay-test-v2/controllers/realtime-turn-controller.ts`
- `admin/components/roleplay-test-v2/controllers/roleplay-ops-controller.ts`
- `admin/components/roleplay-test-v2/controllers/section-view-model.ts`
