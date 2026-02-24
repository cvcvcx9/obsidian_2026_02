---
유형: 개선-프론트엔드
프로젝트: 001_malang
기능: roleplay-test-v2-aggressive-controller-split-phase3
상태: 완료
수정일: 2026-02-20
태그: [flow, improvement, admin, realtime, refactor]
---

# roleplay-test-v2 공격적 컨트롤러 분리 3차

## 배경
- 2차 분리 이후 `page.tsx`에서 가장 큰 로직 덩어리는 상태 스냅샷 반영(`applyStatusSnapshot`)과 auto-silence `useEffect`였다.
- 둘 다 Realtime 오케스트레이션의 하위 정책 로직이라 페이지 엔트리에서 분리해도 동작 변경 없이 축소가 가능했다.

## 적용 내용
- 신규 컨트롤러 추가
  - `admin/components/roleplay-test-v2/controllers/status-snapshot-controller.ts`
    - `applyStatusSnapshotController(status, source, deps)`
    - 슬롯 상태 변경 계산/로그 적재/preview 반영/ref 동기화를 전담
  - `admin/components/roleplay-test-v2/controllers/auto-silence-controller.ts`
    - `setupAutoSilenceAfterQuestionController(params)`
    - auto-silence 조건 분기 + 타이머 등록/해제 + `detect.silence` 이벤트 전송을 전담
- `admin/app/(dashboard)/roleplay-test-v2/page.tsx`
  - 기존 `applyStatusSnapshot` 본문을 컨트롤러 호출로 치환
  - 긴 auto-silence `useEffect` 본문을 컨트롤러 호출로 치환
  - 분리 후 불필요해진 `SlotChangeEntry` import 제거

## 효과
- `roleplay-test-v2/page.tsx` 라인 수 `2152 -> 2082`(약 70라인 축소)
- 1차 대비 누적 축소: `3211 -> 2082`(약 1129라인 축소)
- 상태 반영 정책과 침묵 타이머 정책이 각각 단일 컨트롤러로 수렴되어 추후 변경 시 영향 범위가 명확해짐

## 검증
- LSP diagnostics
  - `admin/app/(dashboard)/roleplay-test-v2/page.tsx`: 없음
  - `admin/components/roleplay-test-v2/controllers/status-snapshot-controller.ts`: 없음
  - `admin/components/roleplay-test-v2/controllers/auto-silence-controller.ts`: 없음
- ESLint(변경 범위)
  - `yarn eslint "app/(dashboard)/roleplay-test-v2/page.tsx" "components/roleplay-test-v2/controllers/status-snapshot-controller.ts" "components/roleplay-test-v2/controllers/auto-silence-controller.ts"`
- Build
  - `yarn build` 통과

## 변경 파일
- `admin/app/(dashboard)/roleplay-test-v2/page.tsx`
- `admin/components/roleplay-test-v2/controllers/status-snapshot-controller.ts`
- `admin/components/roleplay-test-v2/controllers/auto-silence-controller.ts`
