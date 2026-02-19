---
유형: 개선-가이드
프로젝트: 001_malang
기능: realtime-idempotency
상태: 제안
수정일: 2026-02-20
태그: [flow, roleplay, frontend, realtime, idempotency]
---

# idempotency 개선 방향

## 왜 필요한가
- Realtime 환경에서는 같은 의미의 이벤트가 중복 수신될 수 있다.
- 중복 처리되면 `sendFsmEvent`와 `applyFsmActions`가 2번 실행되어 상태 꼬임이 발생한다.

## 어디를 보호해야 하나
- **이벤트 처리**: 같은 이벤트 재수신 시 handler 재실행 방지
- **FSM 전이 호출**: 같은 전이를 1번만 서버로 전송
- **액션 실행**: 같은 action을 1번만 실행
- **맵 정리**: cleanup 함수를 여러 번 호출해도 결과 동일(no-op) 보장

## 구현 방식(쉽게)
1. 이벤트 키를 만든다 (`event_id` 우선, 없으면 대체 키 조합).
2. 최근 처리 키를 `processedEvents`(TTL Map)에 저장한다.
3. 이미 처리된 키면 즉시 return 한다.
4. `sendFsmEvent`에도 `idempotencyKey`를 붙인다.
5. `applyFsmActions`에도 `appliedActionKeys`를 두고 중복 실행을 막는다.

## 간단 예시
```ts
function makeEventKey(event: NormalizedEvent): string {
  if (event.eventId) return event.eventId;
  return [event.type, event.responseId ?? "-", event.itemId ?? "-", event.turnId ?? "-"].join(":");
}

if (processedEvents.has(eventKey)) return; // duplicate skip
processedEvents.set(eventKey, Date.now());
```

```ts
await sendFsmEvent("model.done", payload, {
  idempotencyKey: `model.done:${sessionId}:${responseId}`,
});
```

## 우선 적용 순서
- 1) `sendFsmEvent` idempotencyKey 도입
- 2) `applyFsmActions` 중복 실행 방지
- 3) 이벤트 키 TTL 캐시 도입
- 4) cleanup 멱등 함수 통일

## 완료 체크
- 같은 이벤트 2회 수신 시 실제 부수효과는 1회만 발생
- `done -> canceled` / `canceled -> done` 순서 역전에서도 최종 상태 동일
- 중복 skip 로그(`idempotency skip`)가 확인됨

## 연결 노트
- 상위 가이드: [[01_projects/001_malang/flows/improvements/002_이벤트 분기 집중 개선 방향|이벤트 분기 집중 개선 방향]]
- 맵 정리: [[01_projects/001_malang/flows/improvements/004_리얼타임 목적-응답-아이템 맵 정리 개선안|리얼타임 목적-응답-아이템 맵 정리 개선안]]
