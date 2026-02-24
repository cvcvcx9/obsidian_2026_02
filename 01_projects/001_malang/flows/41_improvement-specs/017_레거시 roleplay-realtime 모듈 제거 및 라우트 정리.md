---
유형: 개선-프론트엔드
프로젝트: 001_malang
기능: roleplay-realtime-legacy-cleanup
상태: 완료
수정일: 2026-02-20
태그: [flow, improvement, admin, cleanup, route]
---

# 레거시 roleplay-realtime 모듈 제거 및 라우트 정리

## 배경
- `/roleplay-test-v2` 도입 이후 구형 `RoleplayRealtimeTester` 경로가 중복 유지되고 있었다.
- `admin/components/roleplay-realtime/` 전체가 레거시 상태였고 타입 빌드 이슈의 원인이 되었다.

## 적용 내용
- `/roleplay-test` 페이지를 구형 컴포넌트 렌더링 대신 `/roleplay-test-v2`로 redirect 하도록 변경했다.
  - 기존 query(`scenarioId`, `prompt`)는 유지 전달.
- 미사용 레거시 코드를 삭제했다.
  - `admin/components/RoleplayRealtimeTester.tsx`
  - `admin/components/roleplay-realtime/**` 전체

## 검증
- `npx eslint "app/(dashboard)/roleplay-test/page.tsx" "app/(dashboard)/roleplay-test-v2/page.tsx" "components/roleplay-test-v2/*.tsx" "components/roleplay-test-v2/context.tsx" "components/roleplay-test-v2/types.ts"`
- `yarn build` 통과

## 변경 파일
- `admin/app/(dashboard)/roleplay-test/page.tsx`
- `admin/components/RoleplayRealtimeTester.tsx` (삭제)
- `admin/components/roleplay-realtime/**` (삭제)
