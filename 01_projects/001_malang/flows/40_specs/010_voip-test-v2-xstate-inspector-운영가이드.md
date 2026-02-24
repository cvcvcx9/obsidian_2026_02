---
유형: 운영가이드
프로젝트: 001_malang
기능: voip-test-v2-xstate-inspector-operations
상태: 운영중
수정일: 2026-02-22
태그: [flow, ops, admin, xstate, inspector]
---

# voip-test-v2 XState Inspector 운영가이드

## 적용 범위
- 대상: `admin/components/voip-test-v2/hooks/use-voip-test-v2-realtime-session-lifecycle.ts`
- 기준: XState v5 + `@xstate/react` + `@statelyai/inspect`

## 현재 구현 요약
- `useMachine(sessionLifecycleMachine, { inspect })` 패턴 사용
- inspector는 모듈 싱글톤(`sharedInspector`)으로 1회 생성
- 개발 환경에서만 inspector 초기화
- inspector 오류는 앱 로직을 깨지 않도록 격리

## 정상 동작 확인 절차
1. `admin`에서 `yarn dev` 실행
2. `/voip-test-v2` 진입
3. 세션 발급 수행
4. Inspector 창에서 `ISSUE_TOKEN_*` 이벤트 확인

## 장애 대응 체크리스트

### 증상 A: Visualizer는 열리는데 이벤트가 없음
1. dev 서버 완전 종료
2. 브라우저의 Stately 관련 탭/팝업 전부 닫기
3. dev 서버 재실행 후 다시 세션 발급

### 증상 B: `invalid state` 메시지 반복
1. 위 A 절차로 세션 초기화
2. 동일 재발 시 브라우저 확장(팝업/보안/광고차단) 비활성화 후 재확인
3. 필요 시 Inspector 연결을 임시 비활성화하고 앱 동작 먼저 검증

### 증상 C: 화면 hydration/런타임 불안정
1. inspector 관련 코드가 렌더 단계에서 브라우저 API를 직접 호출하는지 점검
2. 클라이언트 경계(`"use client"`) + dev 분기 유지 확인

## 운영 원칙
- XState 관련 구현은 v5 공식 문서 기준으로만 적용
- `@xstate/inspect`(v4) 패턴 신규 도입 금지
- inspector는 디버깅 도구이며, 본 로직 실패와 분리해서 다룬다

## 빠른 참조 경로
- 라이프사이클 훅: `admin/components/voip-test-v2/hooks/use-voip-test-v2-realtime-session-lifecycle.ts`
- 메인 설계 문서: `Obsidian Vault/01_projects/001_malang/flows/40_specs/009_roleplay-test-v2-xstate-우선-도입-설계-파인만.md`
