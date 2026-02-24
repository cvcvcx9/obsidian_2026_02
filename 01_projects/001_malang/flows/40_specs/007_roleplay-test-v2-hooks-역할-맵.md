# roleplay-test-v2 hooks 역할 맵

## 목적
- `admin/components/roleplay-test-v2/hooks` 기준으로 파일별 책임과 호출 역할을 빠르게 파악한다.
- `use-roleplay-test-v2-page-model.tsx`에서 어떤 훅/함수 호출이 어떤 관심사를 담당하는지 명시한다.

## 관련 캔버스
- [[01_projects/001_malang/flows/10_overview/008_roleplay-test-v2-전체-흐름-트러블슈팅.canvas|roleplay-test-v2 전체 흐름 트러블슈팅(Canvas)]]
- [[01_projects/001_malang/flows/20_troubleshooting/009_roleplay-test-v2-증상-로그-조치.canvas|roleplay-test-v2 증상-로그-조치(Canvas)]]
- [[01_projects/001_malang/flows/20_troubleshooting/010_roleplay-test-v2-유저-시나리오-이벤트-코드-디버그.canvas|roleplay-test-v2 유저 시나리오 이벤트-코드-디버그(Canvas)]]

## Hook 파일 트리

- `admin/components/roleplay-test-v2/hooks/` 
  - `use-roleplay-test-v2-page-model.tsx` 
  - `use-roleplay-test-v2-state.ts` 
  - `use-roleplay-test-v2-refs.ts` 
  - `use-roleplay-test-v2-session-ops.ts` 
  - `use-roleplay-test-v2-turn-ops.ts` 
  - `use-roleplay-test-v2-realtime-event-model.ts` 
  - `use-roleplay-test-v2-realtime-runtime.ts` 
  - `use-roleplay-test-v2-realtime-session-lifecycle.ts` 
  - `use-roleplay-test-v2-analysis-ops.ts` 
  - `use-roleplay-test-v2-log-and-timers.ts` 
  - `use-roleplay-test-v2-manual-actions.ts` 
  - `use-roleplay-test-v2-section-vm-ports.ts` 
  - `use-roleplay-test-v2-section-vm.ts` 

- `admin/components/roleplay-test-v2/hooks/page-model/` 
  - `deriveRoleplayStatusText.ts` 
  - `useRoleplayTestV2Cleanup.ts` 
  - `useRoleplayTestV2EndConfirmModel.ts` 
  - `useRoleplayTestV2RealtimeMappingController.ts` 
  - `useRoleplayTestV2RealtimeWiring.ts` 
  - `useRoleplayTestV2ScenarioBootstrap.ts` 
  - `useRoleplayTestV2SectionVmInputs.ts` 
  - `useRoleplayTestV2StatusSync.ts` 

## 파일별 책임 요약
### `admin/components/roleplay-test-v2/hooks/use-roleplay-test-v2-page-model.tsx`
- 화면 모델의 메인 오케스트레이션 담당. 상태/refs/ports를 조합하고 하위 훅 호출을 조정해 페이지 모델을 구성한다.
- 최종적으로 페이지 모델 객체를 UI에 노출한다.

### `admin/components/roleplay-test-v2/hooks/use-roleplay-test-v2-state.ts`
- 페이지 상태와 setter 집합 생성.

### `admin/components/roleplay-test-v2/hooks/use-roleplay-test-v2-refs.ts`
- Realtime 연결/턴/매핑 관련 ref 초기화.

### `admin/components/roleplay-test-v2/hooks/use-roleplay-test-v2-session-ops.ts`
- `sendFsmEvent`, `requestHints`, `warmupHints`, `requestAutoHintIfNeeded`를 다루며 서버 FSM/힌트 API와의 연결 담당.

### `admin/components/roleplay-test-v2/hooks/use-roleplay-test-v2-turn-ops.ts`
- `applyFsmActions`, `executeUserTurn`, `requestSlotDetectionForTurn`, `requestStaffTextEnrichment` 등을 수행해 턴 실행 흐름을 관리.

### `admin/components/roleplay-test-v2/hooks/use-roleplay-test-v2-realtime-event-model.ts`
- 이벤트 처리용 deps/ports를 조합해 런타임 훅으로 전달한다.

### `admin/components/roleplay-test-v2/hooks/use-roleplay-test-v2-realtime-runtime.ts`
- 런타임의 라우팅/폴백 분기 및 원시 이벤트 처리를 담당.

### `admin/components/roleplay-test-v2/hooks/use-roleplay-test-v2-realtime-session-lifecycle.ts`
- `issueToken`, `connectRealtime`, `disconnectRealtime` 수명주기를 관리.

### `admin/components/roleplay-test-v2/hooks/use-roleplay-test-v2-analysis-ops.ts`
- 형태소 분석, 통계 로드, 대화 저장 흐름을 담당.

### `admin/components/roleplay-test-v2/hooks/use-roleplay-test-v2-log-and-timers.ts`
- 로그 출력/verbose/guard 로직과 타이머 정리 콜백을 제공.

### `admin/components/roleplay-test-v2/hooks/use-roleplay-test-v2-manual-actions.ts`
- 수동 텍스트/수동 침묵 테스트 액션 담당.

### `admin/components/roleplay-test-v2/hooks/use-roleplay-test-v2-section-vm-ports.ts`
- 섹션 VM 입력(state/actions) 조립 담당.

### `admin/components/roleplay-test-v2/hooks/use-roleplay-test-v2-section-vm.ts`
- 최종 `sectionState`, `sectionActions` 생성 담당.

## use-roleplay-test-v2-page-model.tsx의 고수준 흐름
- 페이지 모델 초기화: `useRoleplayTestV2State()`와 `useRoleplayTestV2Refs()`를 통해 기본 상태/Refs를 구성한다.
- 하위 훅 호출 오케스트레이션: 필요한 포트/종속을 연결하고, 각 훅의 출력을 모아 페이지 모델을 형성한다.
- 외부 UI에 필요한 인터페이스를 구성해 반환한다: `pageModel` 객체/콜백을 제공한다.
- 런타임 상태 변화에 따라 필요한 후속 작업을 트리거한다.

## page-model 내 주요 호출 역할

- `useRoleplayTestV2State()`
- `useRoleplayTestV2Refs()`
- `useRoleplayTestV2LogAndTimers()`
- `useRoleplayTestV2SessionOps(...)`
- `useRoleplayTestV2TurnOps(turnOpsPorts)`
- `useRoleplayTestV2RealtimeEventModel(realtimeEventPorts)`
- `useRoleplayTestV2RealtimeSessionLifecycle(lifecyclePorts)`
- `useRoleplayTestV2ManualActions(...)`
- `useRoleplayTestV2AnalysisOps(...)`
- `useRoleplayTestV2SectionVmPorts(...)`
- `useRoleplayTestV2SectionVm(...)`

## 현재 구조 포인트
- 오케스트레이션(`page-model`)과 실행 로직(ops/runtime/lifecycle)이 분리됨.
- 포트 기반(`context/log/turn/mapping/refs`, `runtime/callbacks/setters/refs`)으로 관심사 경계가 명확해짐.
- 다음 확장 시에는 동일 포트 패턴을 유지해 신규 로직도 같은 레이어로 배치할 것.
