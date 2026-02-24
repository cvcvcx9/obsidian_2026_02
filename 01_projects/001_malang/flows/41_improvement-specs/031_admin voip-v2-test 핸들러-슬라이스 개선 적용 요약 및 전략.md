---
유형: 개선-프론트엔드
프로젝트: 001_malang
기능: admin-voip-v2-test-handler-slice-refactor
상태: 완료
수정일: 2026-02-22
태그: [flow, improvement, admin, voip-v2, refactor, strategy]
---

# admin voip-v2-test 핸들러-슬라이스 개선 적용 요약 및 전략

## 왜 이 작업을 했는가
- `voip-v2-test`는 이벤트 기반 로직이 빠르게 커지면서, 힌트 트리거 시점/중복 요청/스토어 접근 경로가 섞여 가독성과 운영 안정성이 함께 떨어지는 구간이 생겼다.
- 이번 작업은 단순 코드 정리가 아니라, 사용자 체감 흐름(마무리 대사 후 자연스러운 종료), 중복 실행 방지, 운영 중 디버깅 속도 개선을 동시에 달성하기 위한 안정화 작업이다.

## 오늘 적용한 개선 (효과 우선순위)

### 1) 힌트 트리거 단일화
- 문제: 여러 이벤트 경로에서 힌트 요청이 발생해 타이밍 예측이 어려웠다.
- 적용: `transcript.done` 중심으로 힌트 요청 경로를 단일화했다.
- 변경: `admin/components/voip-v2-test/handlers/realtime-transcription-handler.ts`, `admin/components/voip-v2-test/handlers/hint-request-handler.ts`
- 효과: 힌트 요청 시점이 명확해져 회귀 디버깅 범위가 줄었다.

### 2) 힌트 dedupe 키 안정성 강화
- 문제: 턴 경계가 애매한 경우 중복 힌트 요청 가능성이 있었다.
- 적용: timestamp 기반 dedupe 기준으로 턴 구분 안정성을 높였다.
- 변경: `admin/components/voip-v2-test/slices/hint-slice.ts`
- 효과: 동일 발화 기반 중복 요청이 줄어 API 낭비와 UI 깜빡임이 감소한다.

### 3) 힌트 UX 정보 보강
- 문제: 현재 힌트가 어떤 발화/시점/트리거에서 생성됐는지 추적이 어려웠다.
- 적용: `hintSourceUtterance`, `hintRequestTimestamp`, `hintTriggerSource`를 상태/타입/UI에 반영했다.
- 변경: `admin/components/voip-v2-test/types.ts`, `admin/components/voip-v2-test/store.ts`, `admin/components/voip-v2-test/ui/HintSection.tsx`
- 효과: 운영 중 "왜 이 힌트가 떴는지"를 UI에서 즉시 확인할 수 있다.

### 4) 상태 폴링 최적화
- 문제: 탭 비활성 상태에서도 폴링이 유지되어 불필요한 호출이 발생했다.
- 적용: visibility 기반 제어와 backoff를 적용했다.
- 변경: `admin/components/VoipV2ApiTester.tsx`
- 효과: 유휴 상태에서 네트워크/브라우저 비용이 줄고 상태 갱신 효율이 올라간다.

### 5) 스토어 접근 도메인 분리
- 문제: 페이지/컴포넌트에서 store selector가 산재해 수정 영향 범위가 컸다.
- 적용: 세션/리얼타임/힌트/분석 도메인별 custom hooks로 분리했다.
- 변경: `admin/components/voip-v2-test/hooks/use-session.ts`, `admin/components/voip-v2-test/hooks/use-realtime.ts`, `admin/components/voip-v2-test/hooks/use-hint.ts`, `admin/components/voip-v2-test/hooks/use-assistant-analysis.ts`, `admin/components/voip-v2-test/hooks/index.ts`
- 효과: UI 레이어가 "상태 사용" 중심으로 단순화되어 후속 리팩터링 비용이 낮아진다.

### 6) 테스트 기반 확장 준비
- 문제: 테스트 프레임워크 부재로 변경 검증이 수동 중심이었다.
- 적용: 핵심 핸들러/슬라이스 테스트 파일 구조를 선행 배치했다.
- 변경: `admin/components/voip-v2-test/__tests__/hint-slice.test.ts`, `admin/components/voip-v2-test/__tests__/hint-request-handler.test.ts`, `admin/components/voip-v2-test/__tests__/realtime-purpose-handler.test.ts`, `admin/components/voip-v2-test/__tests__/README.md`
- 효과: 프레임워크 도입 즉시 회귀 테스트를 붙일 수 있는 발판을 확보했다.

## 개선 전략 (상세)

### 전략 원칙
- 안정성 우선: 신규 기능보다 중복 실행/순서 역전/의도치 않은 재트리거를 먼저 차단한다.
- 관측 가능성 확보: 모든 비동기 요청은 "트리거 원인 + 시각 + 대상"을 남겨 문제를 재현 가능하게 만든다.
- 경계 분리: 핸들러(이벤트 해석), 슬라이스(상태 전이), UI(표시) 책임을 섞지 않는다.
- 작은 배치: 한 번에 큰 구조 개편보다, 사용자 체감 효과가 큰 축부터 단계적으로 적용한다.
- 검증 동반: lint/build 통과를 기본선으로 두고, 테스트 프레임워크 도입 후 회귀 자동화를 확장한다.

### 다음 적용 순서 (권장)
1. 테스트 러너(vitest) 도입 후 현재 `__tests__` 활성화
   - 목표: 힌트 트리거/중복 차단/턴 경계 회귀 케이스 자동 검증
2. 힌트 요청 메트릭 표준화
   - 목표: trigger source별 성공/실패/중복차단 카운트 수집
3. 종료 전환(ENDED) 체감 안정화
   - 목표: "마무리 대사 -> UI 표시 -> ENDED" 전환 시간 분포를 측정해 UX 일관성 유지
4. store selector 정리 2차
   - 목표: view-model 훅과 action 훅 분리로 렌더 영향 범위 축소

### 리스크와 대응
- 리스크: 이벤트 순서가 환경별로 달라질 때 힌트 요청 누락 또는 중복 발생 가능
- 대응: dedupe 키와 trigger source 로그를 함께 남겨, 누락/중복을 로그 상에서 즉시 구분
- 리스크: 테스트 러너 미도입 상태에서 회귀 탐지 지연
- 대응: 프레임워크 도입 전까지는 변경 파일 단위 lint/build + 수동 시나리오 점검 체크리스트 유지

## 검증 메모
- 코드 반영 후 lint/build 기준 검증을 수행했다.
- 테스트 프레임워크 미설치 상태를 고려해 테스트 파일/구조를 먼저 준비했다.
- 관련 커밋: `b087cc8`

## 연결 노트
- [[01_projects/001_malang/flows/41_improvement-specs/000_개선 방향 인덱스|개선 방향 인덱스]]
- [[01_projects/001_malang/flows/41_improvement-specs/006_개선 실행 체크리스트|개선 실행 체크리스트]]
- [[01_projects/001_malang/flows/41_improvement-specs/030_roleplay-test-v2 공격적 컨트롤러 분리 3차|roleplay-test-v2 공격적 컨트롤러 분리 3차]]
- [[01_projects/001_malang/flows/41_improvement-specs/031_admin voip-v2-test 현재 로직.canvas|admin voip-v2-test 현재 로직 캔버스]]
- [[01_projects/001_malang/flows/41_improvement-specs/032_handler-slice-refactor 스킬 맵.canvas|handler-slice-refactor 스킬 캔버스]]
- [[01_projects/001_malang/flows/41_improvement-specs/033_handler-slice-refactor 스킬 맵.excalidraw|handler-slice-refactor 스킬 Excalidraw]]
