---
유형: 개선-프론트엔드
프로젝트: 001_malang
기능: realtime-message-handler-split
상태: 완료
수정일: 2026-02-20
태그: [flow, improvement, admin, realtime, refactor]
---

# handleRealtimeMessage 책임 분리 2차

## 배경
- `roleplay-test-v2/page.tsx`의 `handleRealtimeMessage`가 이벤트 파싱/중복가드/로깅/이벤트별 비즈니스 처리까지 동시에 담당해 가독성이 떨어졌다.
- 특히 `speech_started`, `input_audio_transcription.completed`, `output_audio_transcript.*` 처리 로직이 길어 유지보수 부담이 컸다.

## 적용 내용
- 이벤트별 세부 처리를 개별 콜백으로 분리했다.
  - `handleSpeechStartedRealtimeEvent`
  - `handleUserTranscriptCompletedRealtimeEvent`
  - `handleAssistantTranscriptDeltaRealtimeEvent`
  - `handleAssistantTranscriptDoneRealtimeEvent`
  - `handleOutputAudioStoppedRealtimeEvent`
- `handleRealtimeMessage`는 아래 3단계만 수행하도록 단순화했다.
  1. 이벤트 정규화/중복가드/로그
  2. 기존 라우터(`routeRealtimeEvent`) 우선 처리
  3. 미라우팅 이벤트를 `eventHandlers` 테이블로 위임

## 효과
- 이벤트 처리 책임 경계가 명확해져 디버깅 포인트가 분리됨
- 신규 이벤트 추가 시 `eventHandlers` 엔트리 추가 중심으로 확장 가능
- 동일 동작 유지하면서 함수 복잡도와 분기 깊이 감소

## 검증
- `npx eslint "app/(dashboard)/roleplay-test-v2/page.tsx"`

## 변경 파일
- `admin/app/(dashboard)/roleplay-test-v2/page.tsx`
