---
goal: setState 호출부터 Function Component 본문이 다시 실행되기 시작할 때까지 React 내부 호출 흐름을 소스코드 기준으로 설명한다
repo: https://github.com/facebook/react
tag: v19.2.0
sha: ae74234eae6ebd62f19190731278e20bc1c37d51
local: ~/Desktop/workspace/react
updated: 2026-08-11
---

# setState → Function Component 재실행

## Units

### 구간 1 — setter에서 Update 객체까지
- [x] [u01 useState가 반환한 setter가 bind하는 내부 함수와 고정 인자](u01-setter-binding.md)
- [ ] u02 dispatchSetState가 lane을 얻는 위치   ← NEXT
- [ ] u03 Update 객체의 생성 위치와 필드 목록
- [ ] u04 Update가 처음 전달되는 함수
- [ ] u05 enqueueConcurrentHookUpdate 반환값 root의 출처

### 구간 2 — 스케줄링 (예정)
- [ ] u06 root → scheduleUpdateOnFiber 연결
- [ ] u07 scheduleUpdateOnFiber → ensureRootIsScheduled
- [ ] u08 Scheduler에 콜백이 등록되는 지점

### 구간 3 — 렌더 시작 (예정)
- [ ] u09 스케줄된 콜백 → performWorkOnRoot
- [ ] u10 workLoop → performUnitOfWork → beginWork
- [ ] u11 beginWork → renderWithHooks → 컴포넌트 함수 호출

## Checkpoint
(없음)

## Parking Lot
- lane은 왜 bitmask인가
- eager bailout은 왜 존재하는가
