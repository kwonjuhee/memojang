## this의 필요성

**상황1. 메서드가 자신이 속한 객체의 프로퍼티를 참조하려면 먼저 자신이 속한 객체를 가리키는 식별자를 참조할 수 있어야 한다.**

다음 예제는 메서드 내부에서 메서드 자신이 속한 객체를 가리키는 식별자를 재귀적으로 참조한다.

메소드가 호출되는 시점에는 이미 객체 리터럴의 평가가 완료되어 객체가 생성되었고 circle 식별자에 생성된 객체가 할당된 이후다. 따라서 메서드 내부에서 circle 식별자를 참조할 수 있다.

=> 하지만 자기 자신이 속한 객체를 재귀적으로 참조하는 방식은 일반적이지 않으며 바람직하지 않다.

```jsx
const circle = {
  radius: 5,
  getDiameter() {
    return 2 * circle.radius;
    // this를 사용해 수정시 return 2 * this.radius
  },
};

console.log(circle.getDiameter()); // 10
```

**상황2. 생성자 함수 방식으로 인스턴스를 생성하는 경우에 생성자 함수 내부에서는 프로퍼티 또는 메서드를 추가하기 위해 자신이 생성할 인스턴스를 참조할 수 있어야 한다.**

=> 하지만 생성자 함수를 정의하는 시점에서는 인스턴스를 생성하기 이전이므로 생성자 함수가 생성할 인스턴스를 가리키는 식별자를 알 수 없다.

```jsx
function Circle(radius) {
  ???.radius = radius;
  // this.radius = radius;
}

Circle.prototype.getDiameter = function () {
  return 2 * ???.radius;
  // return 2 * this.radius;
}

const circle = new Circle(5);
```

      💡 따라서 자신이 속한 객체 또는 자신이 생성할 인스턴스를 가리키는 특수한 식별자가 필요하다.
      이를 위해 자바스크립트는 this라는 특수한 식별자를 제공한다.

## this 키워드

자신이 속한 객체 또는 자신이 생성할 인스턴스를 가리키는 **자기 참조 변수(self-referencing variable)** 이다.
this를 통해 자신이 속한 객체 또는 자신이 생성할 인스턴스의 프로퍼티나 메소드를 참조할 수 있다.

- this는 자바스크립트 엔진에 의해 암묵적으로 생성되며, 코드 어디서든 참조할 수 있다.

- 함수 호출시 arguments 객체와 this가 암묵적으로 함수 내부에 전달된다.

- this가 가리키는 값, 즉 this 바인딩은 **함수 호출 방식에 의해 동적으로 결정**된다.
  - java, c++같은 클래스 기반 언어에서 this는 언제나 클래스가 생성하는 인스턴스를 가리킨다. 하지만 자바스크립트의 this는 함수가 호출되는 방식에 따라 결정된다.

> 🔎 this 바인딩이란?
>
> 바인딩이란 식별자와 값을 연결하는 과정을 의미한다. 예를 들어, 변수 선언은 변수 이름(식별자)과 확보된 메모리 공간의 주소를 바인딩하는 것이다. this 바인딩은 this(키워드로 분류되지만 식별자 역할을 한다)와 this가 가리킬 객체를 바인딩하는 것이다.

```jsx
// 전역에서 this는 전역 객체 window를 가리킨다.
console.log(this); // window

// 일반 함수 내부에서 this는 전역 객체 window를 가리킨다.
// strict mode일 경우 undefined가 바인딩됨.
function square(number) {
  console.log(this); // window
  return number * number;
}
square(1);

// 메소드 내부에서 this는 메소드를 호출한 객체를 가리킨다.
const person = {
  name: "juhee",
  getName() {
    console.log(this); // {name: 'juhee', getName: *f*}
    return this.name;
  },
};
console.log(person.getName());

// 생성자 함수 내부에서 this는 생성자 함수가 생성할 인스턴스를 가리킨다.
function Person(name) {
  this.name = name;
  console.log(this); // Person {name: 'juhee'}
}
const me = new Person("juhee");
```

## 함수 호출 방식과 this 바인딩

| 일반 함수 호출                                             | 전역 객체                          |
| ---------------------------------------------------------- | ---------------------------------- |
| 메소드 호출                                                | 메소드를 호출한 객체               |
| 생성자 함수 호출                                           | 생성자 함수가 생성할 인스턴스      |
| Function.prototype.apply/call/bind 메소드에 의한 간접 호출 | 메소드에 첫번째 인수로 전달한 객체 |

### 1. 일반 함수 호출

기본적으로 this에는 전역 객체가 바인딩된다. 어떠한 함수(중첩 함수, 콜백 함수, ...)라도 일반 함수로 호출하면 함수 내부의 this에는 전역 객체가 바인딩된다.

this는 객체의 프로퍼티나 메소드를 참조하기 위한 자기 참조 변수이므로 일반적으로 객체의 메소드 내부 또는 생성자 함수 내부에서만 의미가 있다.
객체를 생성하지 않는 일반 함수에서 this는 의미가 없다.

strict mode가 적용된 일반 함수 내부의 this에는 undefined가 바인딩된다.

**📌중첩 함수 또는 콜백 함수의 this가 전역 객체에 바인딩 될 때 문제점**

헬퍼 함수의 역할을 하므로 외부 함수의 일부 로직을 대신하는 경우가 많다.
이때 외부 함수인 메소드와 this가 일치하지 않다는 점은 헬퍼 함수로 동작하기 어렵게 만든다.

```jsx
var value = 1;

const obj = {
  value: 100,
  foo() {
    console.log(this); // {value: 100, foo: *f*}

    // 메서드 내에서 정의한 중첩 함수
    function bar() {
      console.log(this); // window -> eww!😑
      console.log(this.value); // 1 > eww!😑
    }

    // 콜백 함수
    setTimeout(function () {
      console.log(this); // window -> eww!😑
      console.log(this.value); // 1 -> eww!😑
    }, 100);
  },
};

obj.foo();
```

**📌this바인딩을 일치시키는 방법**

**1. this 바인딩을 변수에 할당하여 사용하기**

```jsx
var value = 1;

const obj = {
  value: 100,
  foo() {
    // 1. this 바인딩(obj)을 변수 that에 할당한다.
    const that = this;

    // 2. 콜백 함수 내부에서 this 대신 that을 참조한다.
    setTimeout(function () {
      console.log(that.value);
    }, 100);
  },
};

obj.foo();
```

**2. `Function.prototype.bind` 메서드 사용하기**

```jsx
var value = 1;

const obj = {
  value: 100,
  foo() {
    setTimeout(
      function () {
        console.log(this.value); // 100
      }.bind(this),
      100
    );
  },
};

obj.foo();
```

**3. 화살표 함수 사용하기**

```jsx
var value = 1;

const obj = {
  value: 100,
  foo() {
    // 화살표 함수 내부의 this는 상위 스코프의 this를 가리킨다.
    setTimeout(() => console.log(this.value), 100); // 100
  },
};

obj.foo();
```

### 2. 메서드 호출

메서드 내부의 this에는 메소드를 소유한 객체가 아닌 **메소드를 호출한 객체**에 바인딩된다.

> 메서드는 객체에 포함된 것이 아니라 독립적으로 존재하는 별도의 객체이기 때문에 다른 객체의 프로퍼티에 할당할 수도 있고 일반 변수에 할당하여 일반함수로 호출될 수도 있다.

```jsx
const person = {
  name: "Lee",
  getName() {
    // 메소드 내부의 this는 메소드를 호출한 객체에 바인딩된다.
    return this.name;
  },
};
// getName 메소드를 호출한 객체는 person이다.
console.log(person.getName()); // Lee

const anotherPerson = {
  name: "Kim",
};

// getName 메소드를 anotherPerson 객체의 프로퍼티에 할당
anotherPerson.getName = person.getName;
// getName 메소드를 호출한 객체는 anotherPerson이다.
console.log(anotherPerson.getName); // Kim

// getName 메소드를 변수에 할당
const getName = person.getName;
// getName 메소드를 일반 함수로 호출
// 이때 this.name은 window.name과 같다.
console.log(getName()); // ''
// 브라우저 환경에서 window.name은 브라우저 창의 이름을 나타내는 빌트인 프로퍼티이며 기본값은 ''이다.
```

프로토타입 메서드 내부에서 사용된 this도 마찬가지로 해당 메서드를 호출한 객체에 바인딩된다.

```jsx
function Person(name) {
  this.name = name;
}

Person.prototype.getName = function () {
  return this.name;
};

const me = new Person("juhee");
// me가 호출
console.log(me.getName()); // juhee

Person.prototype.name = "jjuhee";
// Person.prototype이 호출
console.log(Person.prototype.getName()); // jjuhee
```

### 3. 생성자 함수 호출

생성자 함수 내부의 this에는 생성자 함수가 미래에 생성할 인스턴스가 바인딩된다.

```jsx
function Circle(radius) {
  this.radius = radius;
  this.getDiameter = function () {
    return 2 * this.radius;
  };
}

const circle1 = new Circle(5);
const circle2 = new Circle(10);

console.log(circle1.getDiameter()); // 10
console.log(circle2.getDiameter()); // 20

// new 연산자와 함께 호출하지 않으면 생성자 함수로 동작하지 않는다.
// 그냥 일반 함수 호출한거
const circle3 = Cicle(15);

console.log(circle3); // undefined (Circle 함수는 반환문이 없으므로 암묵적으로 undefined 반환)

// 일반 함수로 호출된 Circle 내부의 this는 전역 객체를 가리킨다.
console.log(radius); // 15
```

### 4. Function.prototype.apply/call/bind 메소드에 의한 간접 호출

**apply와 call**

```jsx
@param thisArg - this로 사용할 객체
@param argsArray - 함수에게 전달할 인수 리스트의 배열 또는 유사 배열 객체
@returns 호출된 함수의 반환값

Function.prototype.apply(thisArg[, argsApply])

@param {thisArg} - this로 사용할 객체
@param args1, args2, ... - 함수에게 전달할 인수 리스트 (,로 구분된)
@returns 호출된 함수의 반환값

Function.prototype.call(thisArg[, arg1[, arg2[, …]]])
```

apply와 call의 본질적인 기능은 함수를 호출하는 것이다. 함수를 호출하면서 첫 번째 인수로 전달한 특정 객체를 호출한 함수의 this에 바인딩한다.

호출할 함수에 인수를 전달하는 방식만 다를 뿐 동일하게 동작한다.

```jsx
function getThisBinding() {
  console.log(arguments);
  return this;
}

// this로 사용할 객체
const thisArg = { a: 1 };

// getThisBinding 함수를 호출하면서 thisArg를 getThisBinding 함수의 this로 바인딩한다.

// 인수를 배열로 묶어서 전달
console.log(getThisBinding.apply(thisArg, [1, 2, 3]));

// 인수를 쉼표로 구분한 리스트 형식으로 전달
console.log(getThisBinding.call(thisArg, 1, 2, 3));

// Arguments(3) [1, 2, 3, calle: *f*, Symbol(Symbol.iterator): *f*]
// { a: 1 }
```

대표적인 용도는 배열 메소드를 사용할 수 없는 유사 배열 객체에 배열 메소드를 사용하는 경우다.

```javascript
function convertArgsToArray() {
  // arguments 객체를 배열로 변환
  // Array.prototype.slice를 인수 없이 호출하면 배열의 복사본을 생성한다.
  const arr = Array.prototype.slice.call(arguments);

  return arr;
}

convetArgsToArrau(1, 2, 3);
```

**bind**

apply와 call 메소드와 달리 함수를 호출하지 않는다.

첫 번째 인수로 전달한 값으로 this 바인딩이 교체된 함수를 새롭게 생성해 반환한다.

```jsx
function getThisBinding() {
  return this;
}

const thisArg = { a: 1 };

// thisArgs로 this 바인딩이 교체된 getThisBinding 함수를 새롭게 생성해 반환한다.
console.log(getThisBinding.bind(thisArg)); // getThisBinding
// bind는 함수를 호출하지 않으므로 명시적으로 호출해야 한다.
console.log(getThisBinding.bind(thisArg)()); // { a: 1 };
```

메소드의 this와 메소드 내부의 중첩 함수 또는 콜백 함수의 this가 불일치하는 문제를 해결하기 위해 유용하게 사용된다.

```jsx
const person = {
  name: "Lee",
  foo(callback) {
    // 시점1
    setTimeout(callback, 100);
  },
};

// 일반 함수로 호출된 콜백 함수 내부의 this.name은 브라우저 환경에서 window.name과 같다.
person.foo(function () {
  console.log(`Hi! my name is ${this.name}.`); // 시점2 Hi! my name is .
});
```

콜백 함수가 호출되기 이전(1)의 this는 foo 메소드를 호출한 객체, 즉 person 객체를 가리킨다. 그러나 person.foo의 콜백 함수가 일반 함수로서 호출된 시점(2)에서 this는 전역 객체 window를 가리킨다.

위의 콜백 함수는 외부 함수와 this가 상이할 경우 헬퍼 함수로의 역할을 하기 어려워진다. 따라서 bind 메소드를 이용하여 this를 일치시킬 수 있다.

```jsx
const person = {
  name: "Lee",
  foo(callback) {
    setTimeout(callback.bind(this), 100);
  },
};

person.foo(function () {
  console.log(`Hi! my name is ${this.name}.`); // Hi! my name is Lee.
});
```

## 주의할 점

**💡 동일한 함수도 다양한 방식으로 호출될 수 있다.**

```jsx
const foo = function () {
  console.dir(this);
};

// 1. 일반 함수 호출
foo(); // window

// 2. 메소드 호출
const obj = { foo };
obj.foo(); // obj

// 3. 생성자 함수 호출
new foo(); // foo {}

// 4. Function.prototype.apply/call/bind 메소드에 의한 간접 호출
const bar = { name: "bar" };

foo.call(bar); // bar
foo.apply(bar); // bar
foo.bind(bar)(); // bar
```

**💡 렉시컬 스코프와 this 바인딩은 결정 시기가 다르다.**

함수의 상위 스코프를 결정하는 방식인 렉시컬 스코프는 함수 정의가 평가되어 함수 객체가 생성되는 시점에 상위 스코프를 결정한다. 하지만 this 바인딩은 함수 호출 시점에 결정된다.
