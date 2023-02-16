## Render-Blocking CSS

- 브라우저가 외부 스타일시트를 요청하면 렌더 트리 구성이 중단된다.

- 외부 리소스는 메인 스레드가 아닌 백그라운드에서 다운로드되기 때문에 DOM 파싱은 계속된다.

- 스타일시트가 모두 파싱되고 CSSOM이 한번 수정된 후 렌더 트리가 업데이트된다.
  - CSSOM은 점진적으로 구성되지 않는다.
  - cascading 특성 때문에 style이 오버라이딩될 수 있어 렌더 트리가 여러번 재구성되고 새로운 스타일로 repaint될 수 있기 때문
  - 이는 FOUC를 유발하여 사용자 경험을 해칠 수 있다.

- embeded 스타일, inline 스타일 파싱은 parser blocking으로 동작한다.

- CSSOM이 완성될 때까지 자바스크립트 실행이 대기되기 때문에 script blocking이라고도 한다.


## Parser-Blocking Scripts

- 다른 외부 리소스(image, stylesheet, pdf, video 등등)와 달리 자바스크립트를 로드하는 script 태그를 만나면 DOM 파싱이 중단된다. 메인 스레드에서 자바스크립트 파일을 로드하고 실행하기 때문이다.

- DOM 파싱은 일반적으로 메인 스레드에서 발생하기 때문에 자바스크립트가 실행중이라면 스레드가 사용 가능해질 때까지 DOM 파싱이 진행되지 않는다.

- 자바스크립트에서 DOM API를 통해 DOM 요소를 조작할 수 있기 때문에 자바스크립트 파일을 문서 뒤쪽에 배치해야 한다.

- async, defer 속성을 사용하여 DOM 파싱을 중단하지 않고 백그라운드에서 자바스크립트 파일을 로드할 수 있다.


https://sangcho.tistory.com/entry/%EB%B8%8C%EB%9D%BC%EC%9A%B0%EC%A0%80%EC%9D%98-Rendering-3-Rendering-Process-in-browsers?category=740188

https://bitsofco.de/understanding-the-critical-rendering-path/