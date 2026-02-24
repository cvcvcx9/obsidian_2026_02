---
유형: 개선-프론트엔드
프로젝트: 001_malang
기능: roleplay-test-v2-realtime-lifecycle-controller-split
상태: 완료
수정일: 2026-02-20
태그: [flow, improvement, admin, realtime, refactor]
---

# realtime lifecycle controller 분리

## 배경
- `page.tsx` 내부에 Realtime 연결/해제/사전 ref 초기화 로직이 길게 포함되어 있어 컨테이너 집중도가 높았다.
- 상태 모듈 분리 이후에도 연결 수명주기 코드는 엔트리 파일에 남아 있어 추가 분리가 필요했다.

## 적용 내용
- 신규 컨트롤러 모듈 추가: `admin/components/roleplay-test-v2/controllers/realtime-lifecycle-controller.ts`
  - `disconnectRealtimeController`
  - `resetRealtimeRefsBeforeIssueTokenController`
  - `runRealtimeConnectAttempt`
- `page.tsx`에서 아래 콜백을 컨트롤러 호출로 교체
  - `disconnectRealtime`
  - `resetRealtimeRefsBeforeIssueToken`
  - `connectRealtime` 내부 연결 시도 본문
- 결과적으로 페이지는 연결/해제 구현 상세 대신 오케스트레이션 호출 중심으로 단순화되었다.

## 효과
- Realtime 수명주기 로직이 컨트롤러 단위로 분리되어 엔트리 가독성 향상
- 연결 시도 플로우를 함수 경계로 분리해 이후 재시도 정책/메트릭 추가 지점 명확화
- 컨테이너 파일에서 ref 조작 상세 코드 밀도 감소

## 검증
- `yarn eslint "app/(dashboard)/roleplay-test-v2/page.tsx" "components/roleplay-test-v2/controllers/realtime-lifecycle-controller.ts" "components/roleplay-test-v2/page-state.ts" "components/roleplay-test-v2/realtime-event-handlers.ts"`
- `yarn build`

## 변경 파일
- `admin/app/(dashboard)/roleplay-test-v2/page.tsx`
- `admin/components/roleplay-test-v2/controllers/realtime-lifecycle-controller.ts`
