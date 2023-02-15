## 동작 원리

함수 컴포넌트는 리렌더링이 발생하면 함수 자체가 다시 호출된다.

따라서 상태 관리를 위해서는 함수가 다시 호출되었을 때 이전 상태를 기억하고 있어야 한다.
React hook에서는 이를 **클로저**를 통해 해결한다.

> 💡클로저란 함수가 자신이 속한 lexical scope를 기억하여 lexical scope 밖에서 실행될 때도 그 스코프에 접근할 수 있는 특성

### useState / useEffect 구현

대충 이런 느낌

```javascript
/** useState */

const React = (function () {
  let states = [];
  let idx = 0;

  return {
    useState(initialVal) {
      const state = states[idx] || initialVal;
      const _idx = idx;
      const setState = (newVal) => {
        states[_idx] = newVal;
      };

      idx++;
      return [state, setState];
    },
    render(Component) {
      idx = 0;
      const Comp = Component();
      Comp.render();
      return Comp;
    },
  };
})();
```

```javascript
/** useEffect */

function useEffect(callback, depArray) {
  const hasNoDeps = !depArray;
  const deps = states[idx]; // type: array | undefined
  const hasChangedDeps = deps
    ? !depArray.every((el, i) => el === deps[i]) // depArray.some((el, i) => el !== deps[i])
    : true;
  if (hasNoDeps || hasChangedDeps) {
    callback();
    states[idx] = depArray;
  }
  idx++;
}
```

- hook이 호출되면 클로저를 형성한다.

- 클로저의 특성으로 인해 훅이 다시 호출되어도 이전 값을 참조할 수 있다.

## 실제 hook이 참조하는 환경?

```jsx
const hook: Hook = {
  memoizedState: null,

  baseState: null,
  baseQueue: null,
  queue: null,

  next: null,
};
```

- memoizedState
  - 이전 state를 저장하고 있어 useState hook이 다시 호출되어도 이전 state를 참조할 수 있다.
  - 이전 effect를 저장하고 있어 deps를 비교할 수 있다.

- next
  - hook들의 클로저가 참조하는 환경은 next 프로퍼티를 통해 연결리스트 형태로 연결된다.
  - 연결리스트를 따라 이전 값을 참조하고 변경하기 때문에 훅의 실행 순서가 보장되어야 한다.

- queue
  - dispatch: 상태 업데이트
  - last: 배치 업데이트 처리 후 생성

## hook 사용 규칙

**1. 최상위(at the Top Level)에서만 Hook을 호출해야 한다.**

반복문, 조건문 혹은 중첩된 함수 내에서 hook을 호출할 수 없다.

**2. 오직 React 함수 내에서 Hook을 호출해야 합니다**

hook을 일반적인 JavaScript 함수에서 호출할 수 없다. (함수 컴포넌트와 커스텀 hook에서만 가능)

### 💡 hook 규칙을 준수해야하는 이유

컴포넌트가 렌더링 될 때마다 항상 동일한 순서로 hook이 호출되는 것을 보장하기 위함이다.

상태들은 실행 순서대로 배열에 저장되기 때문에 이전 함수 컴포넌트의 실행 순서와 맞지 않는다면 엉뚱한 상태를 참조하고 변경하는 문제가 발생할 수 있다.

https://ko.reactjs.org/docs/hooks-rules.html#only-call-hooks-at-the-top-level

https://www.netlify.com/blog/2019/03/11/deep-dive-how-do-react-hooks-really-work/

https://ingg.dev/hook-work/