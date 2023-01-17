## 동작 원리

함수 컴포넌트는 리렌더링이 발생하면 함수 자체가 다시 호출된다.

따라서 상태 관리를 위해서는 함수가 다시 호출되었을 때 이전 상태를 기억하고 있어야 한다.
React hook에서는 이를 **클로저**를 통해 해결한다.

> 💡클로저란 함수가 자신이 속한 lexical scope를 기억하여 lexical scope 밖에서 실행될 때도 그 스코프에 접근할 수 있는 특성

### useState 구현

<details>
  <summary>🏋️‍♀️</summary>
  
```javascript
function useState(initialVal) {
  let _val = initialVal;

const state = () => \_val;

const setState = (newVal) => {
\_val = newVal;
};

return [state, setState];
}

const [counter, setCounter] = useState(0);
console.log(counter()); // 0
setCounter(1);
console.log(counter()); // 1

````

☠️ state가 getter 함수다.

💡 state를 변수로 바꿔보자.

```javascript
function useState(initialVal) {
  let state = initialVal;

  const setState = (newVal) => {
    state = newVal;
  };

  return [state, setState];
}

const [counter, setCounter] = useState(0);
console.log(counter); // 0
setCounter(1);
console.log(counter); // 0
````

☠️ 변경된 변수 값을 참조할 수 없다.

💡 useState를 통해 생성한 상태를 접근하고 유지하려면 state를 useState 바깥에 선언하자.

```javascript
const React = (function () {
  let _val;

  return {
    useState(initialVal) {
      _val ||= initialVal;

      const setState = (newVal) => {
        _val = newVal;
      };

      return [_val, setState];
    },

    render(Component) {
      idx = 0;
      const Comp = Component();
      Comp.render();
      return Comp;
    },
  };

  return { useState, render };
})();
d;

function Component() {
  const [counter, setCounter] = React.useState(0);

  return {
    render: () => console.log(counter),
    click: () => setCounter(counter + 1),
  };
}

let App = React.render(Counter); // render: { count: 0 }
App.click();
App = React.render(Counter); // render: { count: 1 }
```

☠️ 훅을 여러번 사용하면 하나의 변수에 계속 값을 덮어쓰므로 여러 상태를 가질 수 없다.

💡 state를 배열 형식으로 관리하자.

</details>
  
```javascript
const React = (function () {
  let states = [];
  let idx = 0;

return {
useState(initialVal) {
const state = states[idx] || initialVal;
const \_idx = idx;
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

return { useState, render };
})();

````

### useEffect 구현

```javascript
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
````

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
