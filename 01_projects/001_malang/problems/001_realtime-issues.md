---
유형: 문제-로그
프로젝트: 001_malang
수정일: 2026-02-20
태그: [problem, realtime]
---

# Realtime Issues

## Open
- [ ] Non-Korean input is interpreted as Japanese.  
  Cause: prompt behavior.  
  Plan: improve language control prompt.
- [ ] GPT repeats previously executed steps.  
  Cause: context state is not refreshed correctly; only recent turn is recognized.
- [ ] User hint arrives slowly.  
  Cause: cold start.  
  Current mitigation: send one empty warm-up request.  
  Future option: Redis-backed optimization (lower priority).
- [ ] Asking "이름이 뭐에요?" returns "챗봇이라고 불러달라".

## Closed
- 
