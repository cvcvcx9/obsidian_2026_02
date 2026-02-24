# oh-my-opencode 실전 시나리오

## 1) 버그 수정 시나리오 (가장 자주 씀)

- **탐색**: `glob` + `grep` + `read`로 에러 지점/관련 모듈 찾기
- **분석 보강**: 복잡하면 `task(subagent=explore)`로 패턴 추가 조사
- **수정**: `apply_patch`로 최소 변경
- **검증**: `lsp_diagnostics` -> 관련 테스트/빌드(`bash`)
- **보고**: 무엇을 왜 고쳤는지 + 검증 결과 요약

## 2) 새 기능 추가 (백엔드/NestJS)

- **스코프 분해**: `todowrite`로 단계 나누기 (DTO, service, controller, test)
- **규칙 로드**: `nestjs-best-practices` 스킬 적용
- **구현**: 기존 패턴 맞춰 모듈별 수정
- **검증**: LSP 진단 + 단위테스트 + 빌드
- **완료 기준**: API 동작/에러처리/타입 안정성 모두 확인

## 3) 프론트(UI) 개선 (admin/Next.js)

- **규칙 로드**: `web-design-guidelines`, `vercel-react-best-practices` 필수
- **현황 파악**: 기존 컴포넌트 구조/스타일 토큰 확인
- **개선**: 시각 계층, 접근성(focus/label), 반응형, 로딩/에러 상태 보강
- **검증**: `yarn lint`, `yarn build` + 주요 화면 수동 확인
- **결과**: 디자인 일관성 유지 + UX 개선 포인트 명시

## 4) PR 생성 시나리오 (GitHub)

- **사전 점검**: `git status`, `git diff`, `git log`
- **규칙 로드**: `github-pr-review` 스킬 적용
- **정리**: 변경 목적/영향 범위/테스트 결과를 PR 본문에 구조화
- **실행**: 필요시 push 후 `gh pr create`
- **산출물**: PR URL + 리뷰 포인트 + 리스크/체크리스트 전달

## 5) 외부 라이브러리 도입/이슈 해결

- **외부조사**: `task(subagent=librarian)` + `context7`로 공식 문서/실전 예제 수집
- **내부매핑**: 우리 코드 패턴과 충돌 여부 확인
- **적용**: 최소 침습으로 통합
- **검증**: 타입체크/빌드/회귀 테스트
- **기록**: 왜 이 방식이 최선인지 근거 남김

## 빠른 의사결정 룰

- 단순 수정: 직접 처리(`quick`)
- 파일 여러 개/구조 이해 필요: `explore` 병행
- 외부 지식 필요: `librarian` 먼저
- 난이도 높은 설계/막힘: `oracle` 자문
- 항상 마지막은 **진단 + 테스트 + 빌드**로 닫기

---

## 바로 복붙용 템플릿

### 버그 수정 템플릿

```md
## 버그 수정
- 증상:
- 재현 방법:
- 원인:
- 수정 파일:
- 검증 결과(LSP/테스트/빌드):
- 영향 범위:
```

### PR 본문 템플릿

```md
## Summary
-

## Changes
-

## Validation
- [ ] LSP diagnostics
- [ ] Tests
- [ ] Build

## Risks / Notes
-
```
