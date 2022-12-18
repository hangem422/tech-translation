# 왜 super(props)를 써야할까?

Dan Abramov, [Ppersonal Blog](https://overreacted.io/why-do-we-write-super-props), November 30, 2018

난 Hooks가 새롭게 주목받고 있다 들었습니다. 나는 클래스 컴포넌트의 재미있는 사실을 설명하는 것으로 이 블로그를 시작하고 싶습니다.

이 문제들은 리엑트를 제품에 사용하데 중요하지는 않습니다. 그러나 만약 React가 어떻게 동작하는지 관심이 있다면, 이 주제에 흥미를 느끼게 될 겁니다.

---

나는 `super(props)`를 꽤 많이 사용해왔습니다.

```javascript
class Checkbox extends React.Component {
  constructor(props) {
    super(props);
    this.state = { isOn: true };
  }
  // ...
}
```

물론, [class fields proposal](https://github.com/tc39/proposal-class-fields)을 사용하면 생략할 수 있습니다.

```javascript
class Checkbox extends React.Component {
  state = { isOn: true };
  // ...
}
```

이와같은 문법은 2015년에 React 0.13에서 일반 클래스에대한 지원을 추가하면서 [계획](https://reactjs.org/blog/2015/01/27/react-v0.13.0-beta-1.html#es7-property-initializers)됐습니다. 생성자 함수 정의와 `super(props)` 호출은 언제나 클래스 필드가 인체 공학적 대체제(ergonomic alternative)를 제공하기 전까지의 임시적인 해결책으로 사용되어 왔습니다.

그러나 ES2015의 기능만을 사용했던 예제로 돌아가 봅니다.

```javascript
class Checkbox extends React.Component {
  constructor(props) {
    super(props);
    this.state = { isOn: true };
  }
  // ...
}
```

왜 `super`를 호출할까요? 호출 안 할 수는 없나요? 반드시 호출해야만 한다면, `props`를 넘기지 않고 호출하면 어떻게 될까요? 다른 인자는 없을까요? 해답을 찾아봅시다.

---

자바스크립트에서, `super`는 부모 클래스의 생성자 호출을 의미합니다. (예제에서는, `React.Component`의 구현체를 지칭합니다.)

여기서 중요한 것은, 부모 생성자 함수를 호출하기 전까지 `this`를 사용하지 못한다는 것 입니다. 자바스크립트는 다음 상황을 허용하지 않습니다.

```javascript
class Checkbox extends React.Component {
  constructor(props) {
    // 🔴 아직 `this`를 사용하지 못 합니다.
    super(props);
    // ✅ 이제 사용할 수 있습니다.
    this.state = { isOn: true };
  }
  // ...
}
```

자바스크립트가 `this`에 접근하기 전에 부모 생성자 함수를 반드시 호출하도록 강제하는 데에는 중요한 이유가 있습니다. 다음 계층을 생각해 봅니다.

```javascript
class Person {
  constructor(name) {
    this.name = name;
  }
}

class PolitePerson extends Person {
  constructor(name) {
    this.greetColleagues(); // 🔴 허용되지 않는 이유는 밑에 나옵니다.
    super(name);
  }

  greetColleagues() {
    alert("Good morning folks!");
  }
}
```

`this`가 `super` 이전에 사용될 수 있다고 상상해보세요. 한달후에, `greetColleagues`를 Person의 이름을 메세지에 포함하도록 수정할 수도 있습니다.

```javascript
greetColleagues() {
  alert("Good morning folks!");
  alert("My name is " + this.name + ", nice to meet you!");
}
```

하지만 우리는 `this.greetColleagues()`가 `super()` 이전에 호출되고 있다는 것을 `this.name`을 사용할 때 기억하지 못할 수 있습니다. 허면 `this.name`은 정의되지 않은 상태가 될 겁니다. 보시다시피, 이런 코드에서 위의 상황을 고려하기란 쉽지 않습니다.

이런 위험을 피하고자, Javascript는 생성자 함수에서 `this`를 호출하고 싶을 때, `super`를 먼저 호출하도록 강제합니다. 이런 제한 사항은 클래스로 구현된 React 컴포넌트에서도 동일하게 적용됩니다.

```javascript
  constructor(props) {
    super(props);
    // ✅ 이제부터 `this`를 사용할 수 있습니다.
    this.state = { isOn: true };
  }
```

이런 방식은 우리에게 다른 의문을 남깁니다. 왜 `props`를 넘겨야 할까요?

---

당신은 `super`에게 `props`를 넘기는 것이 기본 `React.Component`에서 `this.props`를 초기화하기 위해서 필요하다고 생각할 겁니다.

```javascript
// Inside React
class Component {
  constructor(props) {
    this.props = props;
    // ...
  }
}
```

틀린것은 아닙니다. 실제로 [그런 역할](https://github.com/facebook/react/blob/1d25aa5787d4e19704c049c3cfa985d3b5190e0d/packages/react/src/ReactBaseClasses.js#L22)을 합니다.

그러나 때로는 `props` 인자를 넘기지 않고 `super()`를 호출해도, `render` 혹은 다른 메서드에서 `this.props`에 접근할 수 있습니다.

어떻게 가능할까요? 생성자 함수 호출 직후에 React가 `props`를 인스턴스에 할당하기 때문입니다.

```javascript
// 리엑트 내부
const instance = new YourComponent(props);
instance.props = props;
```

React가 클래스에대한 지원을 추가할 때, ES6 클래스 하나만 고려한 것이 아닙니다. 가능한 광범위한 클래스 추상화를 지원하는 것이 목표였습니다. 컴포너트를 정희하는 것에 대해서 ClojureScript, CoffeeScript, ES6, Fable, Scala.js, TypeScript와 같은 다른 솔루션에 비해 상대적으로 얼마나 성공적인지는 명확하지 않습니다. 따라서 React는 ES6 클래스가 필요함에도 불구하고 `super()` 호출이 필요한지에 대해 의도적으로 의견을 피했습니다.

그래서 `super(props)` 대신에 `super()`를 호출해도 될까요?

**아직은 혼란이 있기에 아닐 수 있습니다.** 물론, React는 생성자 함수가 실행된 다음 `this.props`를 할당하기는 합니다. 그러나 `this.props`는 `super`가 호출되고 생성자 함수가 끝날 때 까지 여전히 `undefined`입니다.

```javascript
// React 내부
class Component {
  constructor(props) {
    this.props = props;
    // ...
  }
}

// 당신의 코드 내부
class Button extends React.Component {
  constructor(props) {
    super(); // 😬 props를 넘기는 것을 깜빡했다.
    console.log(props); // ✅ {}
    console.log(this.props); // 😬 undefined
  }
  // ...
}
```

만약 이 상황이 생성자 함수에서 호출하는 메서드에서 발생한다면 디버그는 더욱 어렵습니다. **그렇기 떄문에 필요하지 않더라도 항상 `super(props)`에 인자를 넘기는 것을 추천합니다.**

```javascript
class Button extends React.Component {
  constructor(props) {
    super(props); // ✅ We passed props
    console.log(props); // ✅ {}
    console.log(this.props); // ✅ {}
  }
  // ...
}
```

그러면 생성자 함수가 끝나기 전에도 항상 `this.props`가 할당돼 있다고 보장할 수 있습니다.

---

React를 오랫동안 사용한 유저가 궁금할 수 있는 것이 한가지 있습니다.

Context API를 클래스 내부에서 사용할 때(레거시 `contextTypes`와 React 16.6에서 추가된 모던 `contextType` API 모두에서), `context`는 생성자 함수에서 두번째 인자를 받을 수 있습니다.

그런데 왜 우리는 `super(props, context)`와 같이 사용하지 않을까요? 할 수는 있습니다만, context가 자주 사용되지 않기 떄문에 이런 위험을 맞이하는 경우가 적습니다.

클래스 필드 방식으로 사용하면 이런 위험이 대부분 사라집니다. 명시적 생성자가 존재하지 않으면, 모든 인자는 자동으로 전달됩니다. 이것이 필요에 따라 `this.props` 혹은 `this.context`를 참조하기 위해서 `state = {}`와 같은 표현 방식이 허용되는 이유입니다.

Hooks를 사용하면, `super`과 `this` 조차 필요하지 않습니다. 허나 이 주제는 다음에 다루도록 하겠습니다.
