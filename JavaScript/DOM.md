## 문서 객체 모델 (Document Object Model)

텍스트인 웹 문서를 브라우저에 렌더링하려면 웹 문서를 브라우저가 이해할 수 있는 구조로 메모리에 올려야 한다.

따라서 브라우저 렌더링 엔진은 HTML 문서를 파싱하여 브라우저가 이해할 수 있는 자료구조인 DOM을 생성한다.

- HTML 문서의 모든 요소와 요소의 어트리뷰트, 텍스트를 객체로 만들고 **(Node 객체)**
- 요소들의 중첩 관계에 의한 계층 구조를 표현하고 **(트리 자료구조)**
- 각 요소들을 자바스크립트를 통해 동적으로 제어할 수 있는 프로퍼티와 메서드를 제공한다. **(DOM API)**

> 💡 즉 DOM은 Node 객체로 구성된 트리 자료구조이며 Node 타입에 따라 필요한 DOM API를 제공하여 정적인 웹 페이지를 동적으로 변경할 수 있도록 한다.

## Node 객체

HTML 요소는 렌더링 엔진에 의해 파싱되어 DOM을 구성하는 노드 객체로 변환된다.

![Frame 15](https://user-images.githubusercontent.com/62097867/209104979-1f058301-75ed-4c02-baf5-7bc29fc483f8.png)

### 노드 객체의 타입

**1. 문서 노드 (document node)**

DOM 트리의 최상위에 존재하는 루트 노드로서 document 객체를 가리킨다.

요소, 어트리뷰트, 텍스트 노드에 접근하려면 문서 노드를 통해야 한다.
즉, DOM 트리의 노드들에 접근하기 위한 진입점 역할을 한다.

> 🔎 document 객체는 브라우저가 렌더링한 HTML 문서 전체를 가리키는 객체로 전역 객체 window의 document 프로퍼티에 바인딩되어 있어 window.document 또는 document로 참조할 수 있다.

**2. 요소 노드 (element node)**

HTML 요소를 가리킨다. 요소들은 중첩에 의한 부모-자식 관계를 가지며 이를 통해 문서의 구조를 표현한다.

어트리뷰트, 텍스트 노드에 접근하려면 먼저 요소 노드를 찾아 접근해야 한다.

모든 요소 노드는 요소별 특성을 표현하기 위해 HTMLElement 객체를 상속한 객체로 구성된다.

**3. 어트리뷰트 노드 (attribute node)**

HTML 요소의 어트리뷰트를 가리킨다.

부모 노드와 연결되어 있지 않아 요소 노드를 통해서만 접근이 가능하다.

**4. 텍스트 노드 (text node)**

HTML 요소의 텍스트를 표현한다.

요소 노드의 자식 노드이자 리프 노드이며 부모 노드인 요소 노드를 통해서 접근이 가능하다.

✖️ Comment 노드, DOCTYPE을 위한 DocumentType 노드, 복수의 노드를 생성하여 추가할 때 사용하는 DocumentFragment 노드 등 총 12개의 노드 타입이 있다.

### 노드 객체의 상속 구조

노드 객체는 표준 빌트인 객체가 아니라 브라우저 환경에서 추가적으로 제공하는 호스트 객체이지만 자바스크립트 객체이므로
프로토타입에 의한 상속 구조를 갖는다.

공통된 기능일수록 프로토타입 체인의 상위에, 개별적인 고유 기능일수록 하위에 위치하도록 프로토타입 체인을 노드 객체에 필요한 기능, 즉 프로퍼티와 메서드를 제공한다.

![image](https://user-images.githubusercontent.com/62097867/209125089-f65394d7-8e66-4875-be53-acda2b4afd5b.png)

- 모든 노드 객체는 `Object`, `EventTarget`, `Node` 인터페이스를 상속받는다.

  - `EventTarget` 인터페이스 기능을 통해 모든 노드 객체는 이벤트를 발생시킬 수 있다.

    - `EventTarget.addEventListener`, `EventTarget.removeEventListener` 등

  - `Node` 인터페이스 기능을 통해 모든 노드 객체는 트리 자료구조의 노드로서 트리를 탐색하거나 노드 정보를 제공받을 수 있다.
    - `Node.parentNode`, `Node.childNodes`, `Node.previousSibling`, `Node.nextSibling` 등
    - `Node.nodeType`, `Node.nodeName` 등

- 문서 노드는 추가적으로 `Document` `HTMLDocument` 인터페이스를 상속받는다.

- 어트리뷰트 노드는 `Attr`, 텍스트 노드는 `CharacterData` 인터페이스를 상속받는다.

- 요소 노드는 `Element` 인터페이스를 상속받는다. 또한 요소 노드는 추가적으로 `HTMLElement`와 태그의 종류별로 세분화된 `HTMLHtmlElement` `HTMLHeadElement` `HTMLBodyElement` `HTMLUListElement` 등의 인터페이스를 상속받는다.
  - 예를 들면 `input` 객체와 `div` 객체는 모두 style 프로퍼티가 있다. 이처럼 HTML 요소가 갖는 공통적인 기능은 `HTMLElement` 인터페이스가 제공한다.
  - 예를 들면 `input` 객체는 `value` 프로퍼티가 필요하지만 `div` 객체는 필요하지 않다. 따라서 필요한 기능을 제공하는 인터페이스(`HTMLInputElement`, `HTMLDivElement` 등)가 HTML 요소의 종류에 따라 각각 다르다.

**input 요소 노드 객체 예시**

![image](https://user-images.githubusercontent.com/62097867/209125175-a3c60683-15ed-4664-b6fc-b49e7b757cce.png)

input 요소를 파싱하여 객체화한 input 요소 노드 객체는 다양한 특성을 갖는 객체이며
이러한 특성을 나타내는 기능들을 프로토타입 체인에 있는 모든 프로토타입을 상속받아 제공한다.

| input 요소 노드 객체의 특성                                                | 프로토타입을 제공하는 객체 |
| -------------------------------------------------------------------------- | -------------------------- |
| 객체                                                                       | Object                     |
| 이벤트를 발생시키는 객체                                                   | EventTarget                |
| 트리 자료구조의 노드 객체                                                  | Node                       |
| 브라우저가 렌더링할 수 있는 웹 문서의 요소(HTML, XML, SVG)를 표현하는 객체 | Element                    |
| 웹 문서의 요소 중에서 HTML 요소를 표현하는 객체                            | HTMLElement                |
| HTML 요소 중에서 input 요소를 표현하는 객체                                | HTMLInputElement           |
