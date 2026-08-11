---
goal: setState 호출부터 Function Component 본문이 다시 실행되기 시작할 때까지 React 내부 호출 흐름을 소스코드 기준으로 설명한다
repo: https://github.com/facebook/react
tag: v19.2.0
sha: ae74234eae6ebd62f19190731278e20bc1c37d51
local: ~/Desktop/workspace/react
updated: 2026-08-11
---

# setState → Function Component 재실행

종료점은 `renderWithHooks`의 `Component(props, secondArg)` — ReactFiberHooks.js:599.

## Units

### 구간 1 — setState 호출부에서 Update 적재까지
`ReactFiberHooks.js` / `ReactFiberConcurrentUpdates.js`

- [x] [u01 useState가 반환한 setter가 bind하는 내부 함수와 고정 인자](u01-setter-binding.md)
- [x] [u02 dispatchSetState가 lane을 얻는 위치](u02-lane-lookup.md)
- [ ] u03 Update 객체의 생성 위치와 필드 목록   ← NEXT
- [ ] u04 update가 `enqueueConcurrentHookUpdate`로 넘어가는 호출부와 인자 4개
- [ ] u05 `enqueueUpdate`가 update를 실제로 저장하는 자리 (`queue.pending`이 아니다)

**Checkpoint 1** — setCount → update 적재까지 자료 없이 복원
왜: `queue.pending`에 즉시 넣지 않고 모듈 배열에 쌓아두는 이유는?

### 구간 2 — lane 기록과 root 찾기
`ReactFiberConcurrentUpdates.js` / `ReactFiberWorkLoop.js` / `ReactFiberLane.js`

- [ ] u06 `enqueueUpdate`가 lane을 즉시 기록하는 두 필드
- [ ] u07 `enqueueConcurrentHookUpdate`의 반환값 root를 만드는 함수와 순회 방향
- [ ] u08 `scheduleUpdateOnFiber`가 root에 lane을 기록하는 함수와 필드
- [ ] u09 `scheduleUpdateOnFiber` → `ensureRootIsScheduled` 호출 위치

**Checkpoint 2** — lane이 기록되는 세 자리를 흐름 위에 배치
왜: `concurrentQueues` / `fiber.lanes` / `root.pendingLanes`로 나눠 기록하는 이유는?

### 구간 3 — microtask까지
`ReactFiberRootScheduler.js`

- [ ] u10 `ensureRootIsScheduled`가 root를 넣는 자료구조
- [ ] u11 `ensureScheduleIsScheduled`가 microtask를 예약하는 함수
- [ ] u12 microtask 안에서 실행되는 함수와 root 순회
- [ ] u13 `scheduleTaskForRootDuringMicrotask`의 sync / concurrent 분기 조건

**Checkpoint 3** — setState부터 microtask까지 복원
왜: setState가 렌더를 즉시 시작하지 않고 microtask까지 미루는 이유는?

### 구간 4 — 렌더 진입 두 경로
`ReactFiberRootScheduler.js` / `ReactFiberWorkLoop.js`

- [ ] u14 concurrent 분기: `scheduleCallback`에 등록되는 콜백
- [ ] u15 sync 분기: `flushSyncWorkAcrossRoots_impl` → `performSyncWorkOnRoot`
- [ ] u16 `performWorkOnRoot`가 sync/concurrent 렌더를 고르는 조건

**Checkpoint 4** — 두 진입 경로가 같은 함수에서 만나는 지점 확인
왜: 스케줄 우선순위(u13)와 렌더 방식(u16)을 두 번 나눠 판단하는 이유는?

### 구간 5 — 렌더 스택 준비와 큐 flush
`ReactFiberWorkLoop.js` / `ReactFiberConcurrentUpdates.js`

- [ ] u17 `renderRootSync`가 `prepareFreshStack`을 호출하는 조건
- [ ] u18 `prepareFreshStack`이 만드는 workInProgress fiber
- [ ] u19 `prepareFreshStack`에서 `finishQueueingConcurrentUpdates`를 부르는 위치
- [ ] u20 `finishQueueingConcurrentUpdates`가 `queue.pending`을 연결하는 방식
- [ ] u21 `markUpdateLaneFromFiberToRoot`가 세팅하는 필드

**Checkpoint 5** — 적재(구간 1) → flush(구간 5) 사이가 얼마나 떨어져 있는지 복원
왜: return 경로를 `getRootForUpdatedFiber`와 `markUpdateLaneFromFiberToRoot`로 두 번 순회하는 이유는?

### 구간 6 — 워크 루프에서 fiber 하나까지
`ReactFiberWorkLoop.js` / `ReactFiberBeginWork.js`

- [ ] u22 `workLoopSync`의 루프 조건과 호출 대상
- [ ] u23 `performUnitOfWork`가 `beginWork`에 넘기는 인자 3개
- [ ] u24 `beginWork`의 bailout 검사 — `checkScheduledUpdateOrContext`
- [ ] u25 `bailoutOnAlreadyFinishedWork`가 자식으로 내려갈지 판단하는 필드

**Checkpoint 6** — 기록해둔 lane이 회수되는 두 지점 확인
왜: `fiber.lanes`와 `childLanes`를 따로 두는 이유는?

### 구간 7 — 컴포넌트 함수 호출
`ReactFiberBeginWork.js` / `ReactFiberHooks.js`

- [ ] u26 `beginWork`의 FunctionComponent 분기와 `workInProgress.lanes` 초기화
- [ ] u27 `updateFunctionComponent` → `renderWithHooks`
- [ ] u28 `renderWithHooks`의 dispatcher 교체와 `Component(props, secondArg)` 호출
- [ ] u29 (경계 밖 1스텝) `updateState`가 `queue.pending`을 소비하는 위치

**Checkpoint 7 (최종)** — 전체 흐름 복원 + 변형 질문
- 변형 1: 같은 fiber에 `setCount`를 연속 3번 부르면 `concurrentQueues`와 `queue.pending`은 각각 어떻게 되나?
- 변형 2: `setCount`를 이벤트 핸들러 밖(`setTimeout` 콜백)에서 부르면 흐름의 어디가 갈라지나?

## Checkpoint
(없음)

## Parking Lot
- lane은 왜 bitmask인가
- eager bailout은 왜 존재하는가
- `markStarvedLanesAsExpired` — lane starvation 처리
- `renderRootConcurrent`의 yield 판단
- `entangleTransitionUpdate`
- Strict Mode 컴포넌트 이중 호출 (`shouldDoubleRenderDEV`)
- `updateWorkInProgressHook` — 훅 순서가 유지되는 방식

## 계획 점검 이력
- 2026-08-11: 스킬 개정(설계 전 전체 읽기 의무화) 후 11 Unit → 29 Unit 재설계. 구멍 7개:
  1. `finishQueueingConcurrentUpdates` 누락 — `queue.pending`이 setState 시점에 안 채워지는 사실 자체가 빠짐 (ConcurrentUpdates.js:50, WorkLoop.js:2193)
  2. sync 진입 경로 누락 — `flushSyncWorkAcrossRoots_impl` → `performSyncWorkOnRoot` (RootScheduler.js:185, :608)
  3. 순회 2회 구분 없음 — `getRootForUpdatedFiber`(:251) vs `markUpdateLaneFromFiberToRoot`(:188)
  4. `beginWork` bailout 검사 누락 — `checkScheduledUpdateOrContext`(BeginWork.js:3826), `childLanes` 검사(:3732). 기록한 lane이 회수되는 지점
  5. 구 u08이 microtask 예약~우선순위 분기 4단계를 한 줄로 압축
  6. 구 u10에 탐색 edge 3개
  7. 왜 질문·Checkpoint·변형 질문 0개 — 완료 기준 2·4번 통과 장치 없음
