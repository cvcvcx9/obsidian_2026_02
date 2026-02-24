# 008_app-roleplay-realtime-hook-역할 맵

- 모듈 트리(파일 목록)
  - orchestrator: useRealtimeSession.ts
  - lifecycleController.ts
  - messageRouter.ts
  - turnController.ts
  - audioController.ts
  - sessionOps.ts
  - useRealtimeSessionRefs.ts
  - constants.ts, types.ts

- 이벤트 라우팅(이벤트 타입 → handler 파일)
  - session.updated → messageRouter.ts
  - output_audio_buffer.started|stopped|cleared → audioController.ts + messageRouter.ts
  - response.output_audio_transcript.delta|done → messageRouter.ts + turnController.ts
  - conversation.item.input_audio_transcription.completed → messageRouter.ts + turnController.ts
  - error → messageRouter.ts

- 문제별 포인터(최소 5개)
  - useRealtimeSession.ts를 중심으로 세션 생명주기를 orchestrator가 관리하고, 이벤트 수신 핸들러를 연결한다.
  - app/hooks/roleplay/realtime/ 하위 모듈들이 실시간 이벤트를 수신/발행하는 흐름을 보장한다.
  - useRealtimeSessionRefs.ts의 레퍼런스 매핑으로 turnController.ts와 audioController.ts 간의 상태 공유를 확인한다.
  - messageRouter.ts에서 session.updated, error, conversation.item.input_audio_transcription.completed 등 이벤트를 올바른 핸들러로 라우팅하는지 점검한다.
  - response.output_audio_transcript.delta|done 이벤트는 turnController.ts와 messageRouter.ts 간의 텍스트 트랜스크립트 흐름을 트리거한다.
- literal pointers: 
    - app/hooks/roleplay/useRealtimeSession.ts
    - app/hooks/roleplay/realtime/

- 추가 예시
- 마이크가 안 켜짐 → app/hooks/roleplay/realtime/audioController.ts, app/hooks/roleplay/realtime/messageRouter.ts
- AI 전사가 안 쌓임 → app/hooks/roleplay/realtime/turnController.ts, app/hooks/roleplay/realtime/messageRouter.ts
- 세션 토큰 발급 실패 → app/hooks/roleplay/realtime/sessionOps.ts
- 연결이 안 됨(SDP) → app/hooks/roleplay/realtime/lifecycleController.ts
- disconnect 후 리소스 남음 → app/hooks/roleplay/realtime/lifecycleController.ts
