# u01 useState가 반환한 setter가 bind하는 내부 함수와 고정 인자

## Evidence
- `mountStateImpl` — [ReactFiberHooks.js:1912](https://github.com/facebook/react/blob/ae74234eae6ebd62f19190731278e20bc1c37d51/packages/react-reconciler/src/ReactFiberHooks.js#L1912)
  `queue` 객체를 만든다: `{ pending, lanes, dispatch, lastRenderedReducer, lastRenderedState }`
- `mountState` — [ReactFiberHooks.js:1928](https://github.com/facebook/react/blob/ae74234eae6ebd62f19190731278e20bc1c37d51/packages/react-reconciler/src/ReactFiberHooks.js#L1928)
  ```js
  const dispatch = dispatchSetState.bind(null, currentlyRenderingFiber, queue);
  queue.dispatch = dispatch;
  return [hook.memoizedState, dispatch];
  ```
- `dispatchSetState(fiber, queue, action)` — [ReactFiberHooks.js:3599](https://github.com/facebook/react/blob/ae74234eae6ebd62f19190731278e20bc1c37d51/packages/react-reconciler/src/ReactFiberHooks.js#L3599)

## Interpretation
- `useState`가 돌려주는 setter는 `dispatchSetState`의 부분 적용(partial application)이다.
- `bind`의 첫 인자 `null`은 인자가 아니라 `thisArg`. 미리 채워지는 인자는 `fiber`, `queue` 2개.
- 따라서 `setCount(1)`의 `1`은 세 번째 파라미터 `action`으로 들어간다.
- `fiber`/`queue`는 `mountState`가 실행되는 렌더 시점에 고정되고, 호출 시점에 새로 들어오는 값은 `action`뿐이다.

## Unknown
- `currentlyRenderingFiber`는 mount 시점 fiber인데, 이후 렌더에서 `alternate`로 교체되면 bind된 값은 어떻게 처리되는지 (`dispatchSetStateInternal`이 `fiber.alternate`를 보는 이유)
- `queue.lanes`, `queue.pending`이 실제로 언제 채워지는지

## 확인
Q: `setCount(1)` 호출할 때 `1`은 `dispatchSetState`의 몇 번째 파라미터로 들어가나?
A: 세 번째 `action`. `fiber`, `queue`는 `bind`로 이미 채워져 있다.
