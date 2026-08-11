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
경로는 sync 렌더 기준. concurrent 워크루프는 transition·retry 전용이라 범위 밖 (Parking Lot).

## Units

### 구간 1 — setState 호출부
`ReactFiberHooks.js` / `ReactFiberWorkLoop.js` / `ReactEventPriorities.js` / `react-dom-bindings/src/client/ReactDOMUpdatePriority.js`

- [x] [u01 useState가 반환한 setter가 bind하는 내부 함수와 고정 인자](u01-setter-binding.md)
- [x] [u02 dispatchSetState가 lane을 얻는 위치](u02-lane-lookup.md)
- [x] [u03 requestUpdateLane이 lane을 고르는 마지막 한 줄과 우선순위 상수값](u03-request-update-lane.md)
- [ ] u04 Update 객체의 생성 위치와 필드 목록   ← NEXT
- [ ] u05 eager 분기의 조건과 두 갈래 결말

**Checkpoint 1** — setCount 호출부터 update 객체 완성까지 자료 없이 복원
왜: 아직 렌더도 안 했는데 setState 시점에 reducer를 미리 돌리는 이유는?

### 구간 2 — Update 적재와 root 찾기
`ReactFiberConcurrentUpdates.js`

- [ ] u06 update가 `enqueueConcurrentHookUpdate`로 넘어가는 호출부와 인자 4개
- [ ] u07 `enqueueUpdate`가 update를 실제로 저장하는 자리 (`queue.pending`이 아니다)
- [ ] u08 `enqueueUpdate`가 lane을 즉시 기록하는 자리 3곳
- [ ] u09 `enqueueConcurrentHookUpdate`의 반환값 root를 만드는 함수와 순회 방향

**Checkpoint 2** — 적재 시점에 기록되는 것과 미뤄지는 것을 갈라서 복원
왜: update 적재는 미루면서 `fiber.lanes`만 즉시 기록하는 이유는? (u05와 연결된다)

### 구간 3 — root 표시와 microtask 예약
`ReactFiberWorkLoop.js` / `ReactFiberLane.js` / `ReactFiberRootScheduler.js`

- [ ] u10 `scheduleUpdateOnFiber`가 root에 lane을 기록하는 함수와 필드
- [ ] u11 `scheduleUpdateOnFiber` → `ensureRootIsScheduled` 호출 위치
- [ ] u12 `ensureRootIsScheduled`가 root를 넣는 자료구조
- [ ] u13 `ensureScheduleIsScheduled`가 microtask를 예약하는 함수

**Checkpoint 3** — setState부터 microtask 예약까지 복원
왜: setState가 렌더를 즉시 시작하지 않고 microtask까지 미루는 이유는?

### 구간 4 — microtask와 렌더 진입
`ReactFiberRootScheduler.js` / `ReactFiberWorkLoop.js` / `ReactFiberLane.js`

- [ ] u14 microtask 안에서 실행되는 함수와 root 순회
- [ ] u15 `scheduleTaskForRootDuringMicrotask`의 sync / concurrent 분기 조건
- [ ] u16 concurrent 분기: `scheduleCallback`에 등록되는 콜백
- [ ] u17 sync 분기: microtask 말미의 flush 호출 → `performSyncWorkOnRoot`
- [ ] u18 `performWorkOnRoot`가 sync/concurrent 렌더를 고르는 조건 (`includesBlockingLane` 정의까지 열어볼 것)

**Checkpoint 4** — 두 진입 경로가 같은 함수에서 만나는 지점 확인
왜: 우선순위 판단(u15)과 렌더 방식 판단(u18)을 두 번 나눠 하는 이유는?
힌트 위치: ReactFiberRootScheduler.js:560-570, :586-589

### 구간 5 — 렌더 스택 준비와 큐 flush
`ReactFiberWorkLoop.js` / `ReactFiberConcurrentUpdates.js`

- [ ] u19 `renderRootSync`가 `prepareFreshStack`을 호출하는 조건
- [ ] u20 `prepareFreshStack`이 만드는 workInProgress fiber
- [ ] u21 `prepareFreshStack`에서 `finishQueueingConcurrentUpdates`를 부르는 위치
- [ ] u22 `finishQueueingConcurrentUpdates`가 `queue.pending`을 연결하는 방식
- [ ] u23 `markUpdateLaneFromFiberToRoot`가 세팅하는 필드

**Checkpoint 5** — 적재(구간 2)와 flush(구간 5) 사이가 얼마나 떨어져 있는지 복원
왜: return 경로를 `getRootForUpdatedFiber`와 `markUpdateLaneFromFiberToRoot`로 두 번 순회하는 이유는?

### 구간 6 — 워크 루프에서 fiber 하나까지
`ReactFiberWorkLoop.js` / `ReactFiberBeginWork.js`

- [ ] u24 `workLoopSync`의 루프 조건과 호출 대상
- [ ] u25 `performUnitOfWork`가 `beginWork`에 넘기는 인자 3개
- [ ] u26 `beginWork`의 bailout 검사 — `checkScheduledUpdateOrContext`
- [ ] u27 `attemptEarlyBailoutIfNoScheduledUpdate` → `bailoutOnAlreadyFinishedWork`가 자식으로 내려갈지 판단하는 필드
- [ ] u28 `beginWork`가 `workInProgress.lanes`를 비우는 위치

**Checkpoint 6** — 기록해둔 lane이 회수되는 지점들 확인
왜: `fiber.lanes`와 `childLanes`를 따로 두는 이유는?
확인 추가: `root.pendingLanes`는 어디서 회수되나 (ReactFiberRootScheduler.js:406-417)

### 구간 7 — 컴포넌트 함수 호출
`ReactFiberBeginWork.js` / `ReactFiberHooks.js`

- [ ] u29 `beginWork`의 FunctionComponent 분기 → `updateFunctionComponent`
- [ ] u30 `updateFunctionComponent` → `renderWithHooks` 인자 매핑
- [ ] u31 `renderWithHooks`가 dispatcher를 교체하는 위치와 mount/update 판별 기준
- [ ] u32 `renderWithHooks`의 `Component(props, secondArg)` 호출   ← 종료점
- [ ] u33 (경계 밖 1스텝) `updateState`가 `queue.pending`과 `hasEagerState`를 소비하는 위치

**Checkpoint 7 (최종)** — 전체 흐름 복원 + 변형 질문
- 변형 1: 같은 fiber에 `setCount`를 연속 3번 부르면 `concurrentQueues`, `queue.pending`, eager 계산 횟수는 각각 어떻게 되나?
- 변형 2: `setCount`를 이벤트 핸들러 밖(`setTimeout` 콜백)에서 부르면 흐름의 어디가 갈라지고 어디서 다시 합쳐지나?

## Checkpoint
(없음)

## Parking Lot
- lane은 왜 bitmask인가
- `renderRootConcurrent` / `workLoopConcurrentByScheduler` — yield 판단
- `markStarvedLanesAsExpired` — lane starvation 처리
- `entangleTransitionUpdate` (일반 setState에선 no-op)
- root가 스케줄 리스트에서 제거되는 지점 (`processRootScheduleInMicrotask`만 가능)
- `getNextLanes` 내부
- Strict Mode 컴포넌트 이중 호출 (`shouldDoubleRenderDEV`)
- `updateWorkInProgressHook` — 훅 순서가 유지되는 방식
- 이벤트 우선순위 매핑 — `getEventPriority`, `ReactDOMSharedInternals.p` 세팅 지점
- lane은 왜 31비트인가 / `clz32`

## 계획 점검 이력

### 2026-08-11 1차 — 11 Unit → 29 Unit
스킬 개정(설계 전 전체 읽기 의무화) 후 재설계. 구멍 7개:
1. `finishQueueingConcurrentUpdates` 누락 — `queue.pending`이 setState 시점에 안 채워지는 사실 자체가 빠짐 (ConcurrentUpdates.js:50, WorkLoop.js:2193)
2. sync 진입 경로 누락 — `flushSyncWorkAcrossRoots_impl` → `performSyncWorkOnRoot` (RootScheduler.js:185, :608)
3. 순회 2회 구분 없음 — `getRootForUpdatedFiber`(:251) vs `markUpdateLaneFromFiberToRoot`(:188)
4. `beginWork` bailout 검사 누락 — `checkScheduledUpdateOrContext`(BeginWork.js:3826), `childLanes` 검사(:3732)
5. 구 u08이 microtask 예약~우선순위 분기 4단계를 한 줄로 압축
6. 구 u10에 탐색 edge 3개
7. 왜 질문·Checkpoint·변형 질문 0개 — 완료 기준 2·4번 통과 장치 없음

### 2026-08-11 2차 — 29 Unit → 33 Unit
독립 감사 결과 반영. 판정은 "구조는 맞고 소수 추가 필요":
1. **추가 u03** `requestUpdateLane` 내부 — WorkLoop.js:835 `eventPriorityToLane(resolveUpdatePriority())`, `DiscreteEventPriority = SyncLane` / `DefaultEventPriority = DefaultLane`(ReactEventPriorities.js:25,27). 이게 없으면 변형 2를 답할 재료가 없다
2. **추가 u05** eager 분기 — Hooks.js:3649-3689. "지금은 무시"가 틀렸다: idle fiber의 첫 setState면 조건이 항상 참이라 reducer가 매번 실행되고(:3665), 값이 달라도 `hasEagerState`/`eagerState`를 update에 심는다(:3670-3671). Checkpoint 2 왜 질문의 정답(ConcurrentUpdates.js:104-106 주석)이 바로 이 분기이고, u33의 `hasEagerState` 소비(Hooks.js:1521)도 여기 근거
3. **수정 u18** `includesBlockingLane` 정의(Lane.js:679-689)까지 열기 — DefaultLane도 blocking이라 setTimeout setState가 `renderRootSync`로 합류. 변형 2의 마지막 고리
4. **수정** 구 u26·u28에 edge 2개씩 들어있던 것 분리 (현 u28/u29, u31/u32)
5. **수정** u27에 `attemptEarlyBailoutIfNoScheduledUpdate`(BeginWork.js:3845, :4089) 명시, Checkpoint 4에 힌트 위치, Checkpoint 6에 `root.pendingLanes` 회수 지점 추가
6. 곁가지 확인(계획에서 빼는 게 맞음): `renderRootConcurrent`/`workLoopConcurrent`(transition·retry 전용), `entangleTransitionUpdate`(no-op), root 스케줄 리스트 제거

### 2026-08-12 3차 — 계획 변경 없음, 스킬을 고쳤다
u03 수행 중 학습자가 지적: lane이 무엇인지 모르는 상태로 진행하고 있었다.
처음엔 Lane 상수·비트 연산 Unit 2개를 추가해서 35 Unit으로 늘렸다가 되돌렸다 — 그게 증상 대응이었다.

근본 원인 2개를 스킬에서 고쳤다:
1. 스킬이 "무슨 코드를 읽을지"만 설계하게 하고 "읽으려면 뭘 이미 알아야 하는지"는 안 물었다. `lane은 정수 비트 하나다` 같은 사실은 호출 경로 위에 없어서 전체 읽기로도, 독립 감사로도 안 잡힌다
   → 스킬 `## 4. Unit 진행`에 「개념은 먼저 준다」 추가. Unit을 내기 직전 필요한 개념을 한두 줄로 주고, 알면 넘어간다. 개념을 Unit으로 만들지 않는다
2. Unknown을 쌓으라고만 하고 회수 장치가 없었다 (u01 Unknown의 `alternate`가 어느 Unit에도 없었다)
   → Unknown에 `→ u12에서 회수` 또는 `→ Parking Lot` 표기 의무화

Unit 목록은 33개 그대로. lane·비트 연산·microtask·순환 리스트 같은 개념은 해당 Unit 앞에서 한 줄로 준다.
