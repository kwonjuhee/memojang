## Context 등장 배경

컴포넌트 트리 안에서 전역적으로 데이터를 공유하기 위해 도입된 방법

같은 데이터를 컴포넌트 트리 안에서 다양한 레벨의 컴포넌트에 전달해야하는 경우 이를 공통된 상위 요소까지 끌어올린 후 props로 넘겨주어야 했다.

- 거대한 트리가 다시 렌더링되는 문제가 발생할 수 있다.
- prop drilling이 발생할 수 있다.

이때 context를 사용하여 데이터 값이 변할 때마다 하위 컴포넌트에 broadcast하여 많은 컴포넌트가 값을 공유할 수 있다.

## 리렌더링 이슈

하나의 context를 사용하게 되면 context를 구독한 모든 컴포넌트가 리렌더링된다.

### 해결방법

1. Split context
2. memo 또는 useMemo로 리렌더링 방지

https://github.com/facebook/react/issues/15156

https://medium.com/crossplatformkorea/context-api-%EC%96%B4%EB%96%BB%EA%B2%8C-%EC%82%AC%EC%9A%A9%ED%95%B4%EC%95%BC-%ED%95%98%EB%8A%94%EA%B0%80-9ef90247713

### 한계

1의 방식은 context가 계속 추가된다면 Provider hell을 야기할 수 있다.

2의 방식은 불필요한 리렌더링이 일어나는 지점을 찾아 일일이 방어해줘야 한다는 번거로움이 발생한다. 또한 상태값이 변경되었는지 확인하는 비교 연산 비용이 추가된다.

이러한 이슈로 context api는 상태 관리 도구로 사용하기에 한계가 있다. 값이 자주 변경되지 않는 상태를 전역적으로 공유하기 위한 용도로만 사용하는 것이 좋다.

도큐먼트에서 context로 관리하기에 적합한 상태로 추천하는 것은 locale, theme, 유저 정보가 있음. 애초에 prop drilling을 피한 데이터 전달만을 목적으로 도입된듯?
