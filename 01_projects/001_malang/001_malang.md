---
유형: 프로젝트
상태: 진행중
시작일: 2026-02-20
수정일: 2026-02-20
태그: [project, malang]
다음_액션: "컨텍스트 갱신 전략 정의"
---

# 001_malang

## Goal
- Build a stable GPT conversation product with predictable context behavior.

## Status
- Current: Active
- Scope: realtime response quality, context persistence, latency improvements.

## Current Issues
- [[01_projects/001_malang/problems/000_문제 인덱스|문제 인덱스]]
- [[01_projects/001_malang/problems/001_realtime-issues|001_realtime-issues]]

## Decisions
- Keep Redis optimization as lower priority until prompt/context issues are stabilized.

## Next Actions
- [ ] Design and apply context refresh rule to prevent repeated steps.
- [ ] Tighten language prompt constraints for non-Korean handling.
- [ ] Re-check cold start mitigation and measure user hint latency.

## Linked Notes
- Problems: [[01_projects/001_malang/problems/001_realtime-issues|001_realtime-issues]]
- Flows: [[01_projects/001_malang/flows/000_기능 플로우 인덱스|기능 플로우 인덱스]]
- Logs: [[01_projects/001_malang/logs/000_로그 인덱스|로그 인덱스]]
- Decisions: [[01_projects/001_malang/decisions/000_결정사항 인덱스|결정사항 인덱스]]
