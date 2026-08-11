# u02 dispatchSetState가 lane을 얻는 위치

## Evidence
- `requestUpdateLane(fiber)` — [ReactFiberHooks.js:3616](https://github.com/facebook/react/blob/ae74234eae6ebd62f19190731278e20bc1c37d51/packages/react-reconciler/src/ReactFiberHooks.js#L3616)
  `dispatchSetState` 본문의 첫 실질 코드. 인자는 bind로 고정된 `fiber`.
- 반환된 `lane`이 쓰이는 곳 3군데 — 같은 파일
  - `dispatchSetStateInternal(fiber, queue, action, lane)` [:3621](https://github.com/facebook/react/blob/ae74234eae6ebd62f19190731278e20bc1c37d51/packages/react-reconciler/src/ReactFiberHooks.js#L3621)
  - `startUpdateTimerByLane(lane, 'setState()', fiber)` [:3624](https://github.com/facebook/react/blob/ae74234eae6ebd62f19190731278e20bc1c37d51/packages/react-reconciler/src/ReactFiberHooks.js#L3624)
  - `markUpdateInDevTools(fiber, lane, action)` [:3626](https://github.com/facebook/react/blob/ae74234eae6ebd62f19190731278e20bc1c37d51/packages/react-reconciler/src/ReactFiberHooks.js#L3626)
- `const update = { lane, ... }` — [ReactFiberHooks.js:3635](https://github.com/facebook/react/blob/ae74234eae6ebd62f19190731278e20bc1c37d51/packages/react-reconciler/src/ReactFiberHooks.js#L3635)

## Interpretation
- lane은 setter 호출 시점에 결정된다. `Update` 객체 생성보다 앞이므로, 객체를 만들 때 `lane`은 이미 확정된 값으로 박힌다.
- 3군데 중 흐름상 필수는 `dispatchSetStateInternal` 하나. 나머지 둘은 계측(update timer)·DevTools 표시용이라 빼도 업데이트는 동작한다.

## Unknown
- `requestUpdateLane`이 lane을 어떤 기준으로 고르는지 (transition 여부, 이벤트 우선순위) → u03에서 회수 완료
- 결정된 lane이 실제로 언제 소비되는지 → u15(스케줄 분기), u18(렌더 방식), u26(bailout 검사)에서 회수

## 확인
Q: `lane` 결정(3616)과 `Update` 객체 생성(3635) 중 먼저 실행되는 쪽은?
A: lane 결정이 먼저. 그래서 `update` 객체는 `{ lane, ... }`로 확정된 값을 받는다.
