---
유형: 개선-안정화
프로젝트: 001_malang
기능: runtime-cache-invalidate-and-slot-fallback
상태: 완료
수정일: 2026-02-20
태그: [flow, improvement, cache, slot-nlu, fsm]
---

# runtime 캐시 무효화 및 수량/스킵 오탐 보강

## 배경
- runtime config를 발행해도 `slotValuePolicyMap`이 즉시 반영되지 않는 케이스가 있었다.
- 수량 인식 실패 후 일반 표현(`괜찮`)이 skip 의도로 오탐되어 다음 슬롯으로 넘어가는 문제가 있었다.

## 원인
- `ScenarioRuntimeService`가 메모리 캐시를 사용하지만 runtime 저장 경로에서 캐시 무효화를 하지 않았다.
- FSM skip intent 키워드가 과도하게 넓어(`괜찮`, `추천` 등) 명시적 스킵이 아닌 문장도 skip으로 처리됐다.
- Slot NLU가 실패하면 quantity/payment를 보조 추출할 fallback이 없었다.

## 변경 내용
- runtime 저장 성공 시 `ScenarioRuntimeService.invalidate()`를 호출하도록 연결했다.
- admin roleplay-scenarios 모듈에 `ScenarioRuntimeModule`을 주입했다.
- `looksLikeSkipIntent`를 명시적 스킵 표현 중심으로 축소했다.
- `SlotNluService`에 heuristic fallback을 추가했다.
  - quantity: `두 잔`, `2개` 등 수량 패턴 추출
  - payment: `수표/카드/현금/페이` 토큰 추출

## 기대 효과
- 정책 저장 후 새 세션에서 즉시 최신 runtime 정책이 반영된다.
- 수량 인식이 실패해도 기본 수량 패턴은 보완 추출된다.
- `카드로 괜찮아요` 같은 표현에서 현재 슬롯이 의도치 않게 skip되지 않는다.

## 검증
- `yarn test -- src/v2/fsm/fsm.service.spec.ts`
- `yarn test -- src/v2/session/slot-nlu.service.spec.ts`
- `yarn test -- src/v2/session/session.service.spec.ts`
- `yarn build`

## 변경 파일
- `server/src/admin/roleplay-scenarios/roleplay-scenarios.module.ts`
- `server/src/admin/roleplay-scenarios/roleplay-scenarios.service.ts`
- `server/src/v2/fsm/fsm.service.ts`
- `server/src/v2/fsm/fsm.service.spec.ts`
- `server/src/v2/session/slot-nlu.service.ts`
- `server/src/v2/session/slot-nlu.service.spec.ts`

## 연결 노트
- [[01_projects/001_malang/flows/41_improvement-specs/014_슬롯 값 정책 맵 추가 및 비허용 값 차단|014_슬롯 값 정책 맵 추가 및 비허용 값 차단]]
