# u03 requestUpdateLane이 lane을 고르는 마지막 한 줄과 우선순위 상수값

## Evidence
- `requestUpdateLane` — [ReactFiberWorkLoop.js:792](https://github.com/facebook/react/blob/ae74234eae6ebd62f19190731278e20bc1c37d51/packages/react-reconciler/src/ReactFiberWorkLoop.js#L792)
  마지막 줄 [:835](https://github.com/facebook/react/blob/ae74234eae6ebd62f19190731278e20bc1c37d51/packages/react-reconciler/src/ReactFiberWorkLoop.js#L835) — `return eventPriorityToLane(resolveUpdatePriority());`
  반환값은 호출부 [ReactFiberHooks.js:3616](https://github.com/facebook/react/blob/ae74234eae6ebd62f19190731278e20bc1c37d51/packages/react-reconciler/src/ReactFiberHooks.js#L3616)의 `lane` 변수에 담긴다
- `EventPriority` 타입 정의 — [ReactEventPriorities.js:22](https://github.com/facebook/react/blob/ae74234eae6ebd62f19190731278e20bc1c37d51/packages/react-reconciler/src/ReactEventPriorities.js#L22)
  `export opaque type EventPriority = Lane;`
- `DiscreteEventPriority = SyncLane` — [ReactEventPriorities.js:25](https://github.com/facebook/react/blob/ae74234eae6ebd62f19190731278e20bc1c37d51/packages/react-reconciler/src/ReactEventPriorities.js#L25)
- `DefaultEventPriority = DefaultLane` — [ReactEventPriorities.js:27](https://github.com/facebook/react/blob/ae74234eae6ebd62f19190731278e20bc1c37d51/packages/react-reconciler/src/ReactEventPriorities.js#L27)
- `eventPriorityToLane` — [ReactEventPriorities.js:51](https://github.com/facebook/react/blob/ae74234eae6ebd62f19190731278e20bc1c37d51/packages/react-reconciler/src/ReactEventPriorities.js#L51)
  본문이 한 줄, `return updatePriority;`
- `resolveUpdatePriority` — [ReactDOMUpdatePriority.js:35](https://github.com/facebook/react/blob/ae74234eae6ebd62f19190731278e20bc1c37d51/packages/react-dom-bindings/src/client/ReactDOMUpdatePriority.js#L35)
  분기 3개: `ReactDOMSharedInternals.p`가 `NoEventPriority`가 아니면 그 값 / `window.event === undefined`면 `DefaultEventPriority` / 그 외 `getEventPriority(currentEvent.type)`

## Interpretation
- `eventPriorityToLane`은 변환 함수가 아니라 identity다. `EventPriority`가 `Lane`의 별칭(`opaque type`)이라 값은 이미 lane이고, 이 함수는 `opaque` 타입 벽을 넘게 해주는 통로 역할만 한다.
- 따라서 "우선순위를 lane으로 바꾼다"는 표현은 부정확하다. 우선순위와 lane은 같은 값 공간을 쓴다.
- lane은 fiber나 컴포넌트의 성질이 아니라 **setState를 부른 시점의 실행 문맥**에서 결정된다. 판단 재료는 진행 중인 이벤트(`window.event`)와 React가 세팅해둔 `ReactDOMSharedInternals.p`.
- 같은 컴포넌트의 같은 setter라도 클릭 핸들러 안에서 부르면 `SyncLane`, `setTimeout` 안에서 부르면 `DefaultLane`이 된다.

## Unknown
- `getEventPriority(currentEvent.type)`가 이벤트 타입별로 우선순위를 어떻게 매핑하는지 → Parking Lot
- `ReactDOMSharedInternals.p`가 언제 세팅되는지 (React DOM 이벤트 진입 지점) → Parking Lot
- transition 분기 (ReactFiberWorkLoop.js:813-833) → Parking Lot
- 결정된 lane이 실제로 소비되는 지점 → u15(스케줄 분기), u18(렌더 방식), u26(bailout 검사)에서 회수

## 확인
Q: `setTimeout` 콜백에서 `setCount(1)`을 부르면 `lane`에 담기는 값은?
A: `DefaultLane`. `window.event`가 `undefined`라 `resolveUpdatePriority`가 `DefaultEventPriority`를 반환하고, `eventPriorityToLane`이 그대로 통과시킨다.
