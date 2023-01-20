## JSX

JSX는 JavaScript를 확장한 문법으로 React 엘리먼트를 생성한다.

```jsx
const element = <h1 className="greeting">Hello, world!</h1>;
```

JSX는 브라우저가 이해할 수 없는 문법이다.
따라서 JSX 문법을 사용하려면 트랜스파일 과정을 거쳐서 **브라우저가 이해할 수 있는 순수한 자바스크립트 코드로 변환**해주어야 한다.
이때 Babel이라는 트랜스파일링 도구를 사용한다.

## 트랜스파일링 결과

### React17 이전

JSX를 `React.createElement()`를 호출하여 컴파일한다.
[여기서 컴파일해볼 수 있음](https://babeljs.io/repl/#?browsers=defaults%2C%20not%20ie%2011%2C%20not%20ie_mob%2011&build=&builtIns=false&corejs=3.21&spec=false&loose=false&code_lz=GYVwdgxgLglg9mABACQKYBt1wBQEpEDeAUIohAgM5SIAWAhmACbqoDC6MEA1ogLyJ4-APkIlSiOiwBOUbAHIA7nCnpGc3GIC-YqaiggpSbGNIAeAEYgoUBIgTtOXXgXpMWD7pqEQO3UwHpLawQhMQ1tIlBIWFsAQQAHeMFiUl19Q0RTNEw4RH8hAG4iTQKgA&debug=false&forceAllTransforms=false&shippedProposals=false&circleciRepo=&evaluate=false&fileSize=false&timeTravel=false&sourceType=module&lineWrap=true&presets=env%2Creact%2Cstage-2&prettier=false&targets=&version=7.20.12&externalPlugins=&assumptions=%7B%7D)

```jsx
const element = React.createElement(
  "h1",
  { className: "greeting" },
  "Hello, world!"
);
```

하지만 이 방식은...

- JSX가 React의 메서드를 사용하여 컴파일되기 때문에 React가 스코프 내에 존재해야 했다.
- [성능 개선과 단순화를 제한](https://github.com/reactjs/rfcs/blob/createlement-rfc/text/0000-create-element-changes.md#motivation)

### React17 부터 도입된 새로운 변환 방식

JSX를 react/jsx-runtime의 `jsx 함수`를 통해 변환한다.

빌드 시점에 Babel이 import문을 주입하기 때문에 직접 import할 필요가 없다.

```jsx
// Inserted by a compiler (don't import it yourself!)
import { jsx as _jsx } from "react/jsx-runtime";

function App() {
  return _jsx("h1", { children: "Hello world" });
}
```

💡 React.createElement를 사용하지 않기 때문에 더이상 import React하지 않아도 된다.

https://ko.reactjs.org/blog/2020/09/22/introducing-the-new-jsx-transform.html

## React 엘리먼트 객체 생성

```jsx
// 주의: 다음 구조는 단순화되었습니다
const element = {
  type: "h1",
  props: {
    className: "greeting",
    children: "Hello, world!",
  },
};
```

## React 엘리먼트 렌더링하기

1. 루트 DOM 엘리먼트를 `ReactDOM.createRoot()`에 전달한다.

2. React 엘리먼트를 `root.render()`에 전달한다.

```jsx
const root = ReactDOM.createRoot(document.getElementById("root"));

root.render(<h1>Hello, world</h1>);
```

이미 렌더링된 엘리먼트는 불변객체이기 때문에 엘리먼트를 생성한 이후에는 해당 엘리먼트의 자식이나 속성을 변경할 수 없다.

따라서 UI를 업데이트하는 유일한 방법은 새로운 엘리먼트를 생성하고 이를 render에 다시 전달하는 것이다.

ReactDOM은 해당 엘리먼트와 이전 엘리먼트를 비교하여 변경된 부분만 DOM에 업데이트한다.
