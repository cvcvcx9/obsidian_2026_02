---
유형: 개선-설계
프로젝트: 001_malang
기능: handle-realtime-message-next-shrinking
상태: 제안
수정일: 2026-02-20
태그: [flow, improvement, admin, realtime, refactor]
---

# handleRealtimeMessage 추가 축소 아이디어

## 목적
- 현재 `handleRealtimeMessage`는 1차 책임 분리 후에도 "공통 파이프라인 + 핸들러 디스패치"를 한 함수에 같이 보유한다.
- 다음 단계에서는 함수 길이보다 **인지 부하(읽을 때 필요한 문맥 수)**를 줄이는 방향이 더 효과적이다.

## 현재 남은 복잡도
- 공통 파이프라인(파싱, dedupe, verbose log, 라우터 처리)이 길다.
- 이벤트 핸들러 맵이 콜백 내부에 생성되어 시각적으로 밀도가 높다.
- `ref` 기반 상태(`speechStartedAtRef`, `assistantBufferRef` 등) 의존이 많아 테스트 분리가 어렵다.

## 축소 전략 제안

### 1) 파이프라인 함수 4단계 고정
- `parseIncomingRealtimeEvent(raw)`
- `guardDuplicateRealtimeEvent(eventType, payload)`
- `logRealtimeEvent(eventType, raw)`
- `dispatchRealtimeEvent(eventType, payload)`

효과:
- `handleRealtimeMessage` 본문은 15~25줄 수준으로 축소 가능
- 디버깅 시 어느 단계에서 반환됐는지 바로 판단 가능

### 2) 디스패치 테이블 외부 상수화(메모이즈)
- 현재 콜백 내부 `eventHandlers`를 `useMemo` 기반으로 올리거나, 핸들러 팩토리 함수로 분리
- 예: `const realtimeEventHandlers = useMemo(() => ({ ... }), [deps])`

효과:
- 본문에서 분기 선언 블록 제거
- 의존성 목록이 이벤트 처리 단위로 분리되어 리뷰가 쉬움

### 3) ref 의존 로직을 작은 서비스 객체로 래핑
- 예: `speechGuardService`, `assistantTranscriptService`
- 서비스가 ref를 캡슐화하고, page는 메서드만 호출

효과:
- page에서 ref 세부 필드를 몰라도 동작
- 회귀 테스트 작성 시 모킹 단위 명확화

### 4) 조기 종료 사유를 enum-like 키로 표준화
- 현재 로그 문자열 기반 사유를 키 기반으로 통합
- 예: `EARLY_RETURN_REASON.DUPLICATE_EVENT`, `EARLY_RETURN_REASON.EMPTY_TRANSCRIPT`

효과:
- 로그 집계/검색 자동화 가능
- 코드 리뷰 시 분기 품질 기준 일관성 확보

### 5) 이벤트 처리 책임 레이어 2분리
- A 레이어: Realtime transport 이벤트(`speech_started`, `output_audio_buffer.stopped`)
- B 레이어: Conversation semantic 이벤트(`transcription.completed`, `response.done`)

효과:
- 이벤트 유형 추가 시 어느 레이어에 넣어야 하는지 명확
- 기능 변화와 인프라 변화 분리 가능

## 권장 실행 순서
1. 파이프라인 4단계 함수 추출(저위험)
2. 핸들러 테이블 외부화(useMemo)
3. 조기 종료 키 표준화
4. ref 래핑 서비스 도입(중위험)
5. 레이어 분리(중위험)

## 체크 포인트
- 단계별 리팩터링 후에도 아래 동작 동일성 유지 필요
  - 이벤트 dedupe
  - `speech_started` 누락 fallback
  - hint auto trigger
  - `response.done` 목적별 처리 및 map cleanup

## 예상 효과
- `page.tsx` 내 실질 이벤트 엔트리 함수 복잡도 추가 20~30% 축소
- 추후 신규 이벤트 추가 시 코드 변경 위치 1~2곳으로 제한
