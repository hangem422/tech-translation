# How Does React Tell a Class from a Function?

Dan Abramov, [Ppersonal Blog](https://overreacted.io/how-does-react-tell-a-class-from-a-function), December 2, 2018

아래 함수로 정의된 `Greeting` 컴포넌트의 경우를 새각해봅시다.

```tsx
function Greeting() {
  return <p>Hello</p>;
}
```

React는 동일한 컴포넌트를 클래스로 정의하는 방법 또한 제공합니다.

```tsx
class Greeting extends React.Component {
  render() {
    return <p>Hello</p>;
  }
}
```

(최근까지는 `state`와 같은 기능을 사용할 수 있는 유일한 방법이였습니다.)

`<Greeting />`을 랜더하고자 할 때, 이것이 어떤 방법으로 구현돼있는지 신경쓰지 않아도 됩니다.

```tsx
// 클래스 혹은 함수 중 어떤것으로 정의돼있든 관계없다.
<Greeting />
```

하지만 React 자체는 이 차이를 고려합니다.

만약 `Greeting`이 함수라면, React는 다음과 같이 호출합니다.

```tsx
// 당신의 코드
function Greeting() {
  return <p>Hello</p>;
}

// React 내부
const result = Greeting(props); // <p>Hello</p>
```

하지만 만약 `Greeting`이 클래스라면, React는 우선 `new` 연산자로 인스턴스를 생성하고 새성된 인스턴스의 `render` 메서드를 사용해야 합니다.

```tsx
// 당신의 코드
class Greeting extends React.Component {
  render() {
    return <p>Hello</p>;
  }
}

// React 내부
const instance = new Greeting(props); // Greeting {}
const result = instance.render(); // <p>Hello</p>
```

두가지 경우 모두 React는 Node를 생성하고자 합니다. 하지만 구체적인 단계는 `Greeting`이 어떻게 구현되었는지에 따라 달라집니다.

그렇다면 React는 어떤 컴포넌트가 함수 혹은 클래스로 구현됐는지 파악할 수 있을까요?

내 [이전 포스트](https://overreacted.io/why-do-we-write-super-props/)에서 언급했듯, React를 생산적으로 사용하기 위해 이 내용을 알 필요는 없습니다. 부디 이걸 인터뷰 질문에 넣지 말아주세요. 사실 React 보다는 Javascript에 대한 내용에 더 가깝습니다.

이 블로그는 React가 특정 방식으로 동작하는 이유가 궁금한 독자들을 위한 블로그입니다. 바로 당신인가요? 그럼 같이 파고들어 봅시다.

긴 여정이 될겁니다. 안전 벨트 매세요. 그 포스트는 React 자체에 대한 정보를 많이 담고있지 않으나, `new`, `this`, `class`, 애로우 함수, `prototype`, `__proto__`, `instanceof`의 몇 가지 측면과 JavaScript에서 함께 작동하는 방식을 살펴봅니다. 다행이도, React를 사용할 떄 위 내용에 대해 고민할 필요가 업습니다. 그래도 만약 당신이 리엑트를 실행중이라면...

(만약 정답만을 알고 싶다면, 맨 아래로 스크롤하세요.)

---

첫번쨰로, 함수와 클래스를 다르게 취급해야 하는 이유를 알 필요가 있습니다. 클래스를 후출할 때 `new` 연산자 어떻게 사용하는지에 주목해보세요.

```tsx
// 만약 Greeting이 함수라면
const result = Greeting(props); // <p>Hello</p>

// 만약 Greeting이 클래스라면
const instance = new Greeting(props); // Greeting {}
const result = instance.render(); // <p>Hello</p>
```

자바스크립트에서 `new` 연산자가 하는 일을 간략하게 알아봅시다.

---

옛날에, 자바스크립트는 클래스가 없었습니다. 하지만, 일반적인 함수로 클래스와 유사한 패턴을 표현할 수 있었습니다. 어떤 함수든 호출하기 전에 `new` 연산자를 붙이면 클래스 생성자와 유사한 역할로 사용할 수 있습니다.

```typescript
// 단순한 함수
function Person(name) {
  this.name = name;
}

var fred = new Person("Fred"); // ✅ Person {name: 'Fred'}
var george = Person("George"); // 🔴 정상 동작하지 않습니다.
```

오늘날에도 여전히 이런 방식으로 코드를 작성할 수 있습니다! DevTools에서 사용해보세요.

만약 `Person('Fred')`를 `new` 없이 사용한다면, 함수 내부의 `this`는 전역적인 무언가를 가리키거나 아무것도 가르키지 않습니다. (예를 들어, `window` 혹은 `undefined`). 하여 우리 코드는 깨지거나, `window.name`에 값을 할당하는 것과 같은 오동작을 일으킵니다.

호출 전에 `new`를 붙이는 것은 다음과 같이 말하는 것과 동일합니다. "자바스크립트야, `Person`이 일반 함수라는 것은 알지만, 클래스의 생성자라고 간주하자. `{}` 객체를 생성하고 `Person` 내부의 `this`가 이 객체를 가르키게 함으로서 나는 `this.name`과 같은 것을 할당할 수 있어. 그리고나서 객체를 내게 반환해줘"

이게 바로 new 연산자가 하는 일입니다.

```typescript
var fred = new Person("Fred"); // `Person` 내부의 `this`와 동일하다.
```

또한 `new` 연산자는 `Person.prototype`에 넣었던 모든 것을 `fred` 객체에서 사용할 수 있게 해줍니다.

```typescript
function Person(name) {
  this.name = name;
}
Person.prototype.sayHi = function () {
  alert("Hi, I am " + this.name);
};

var fred = new Person("Fred");
fred.sayHi();
```

이것이 자바스크립트에 클래스가 직접 주가되기 전에 사람들이 클래스를 모방했던 방법입니다.

> [Javascript의 this에 대해 좀 더 알아보기](https://velog.io/@hangem422/js-this)

> [Javascript의 Prototype에 대해 좀 더 알아보기](https://velog.io/@hangem422/js-prototype)

---

위와 같은 이유로 자바스크립트에서는 한동안 `new`가 사용됐습니다. 하지만 최근에 class가 등장했습니다. 이는 우리가 우리의 의도와 좀 더 가깝도록 코드를 작성할 수 있게 해줬습니다.

```typescript
class Person {
  constructor(name) {
    this.name = name;
  }
  sayHi() {
    alert("Hi, I am " + this.name);
  }
}

let fred = new Person("Fred");
fred.sayHi();
```

개발자의 의도를 파악하는 것은 언어 및 API 디자인에서 중요합니다.

함수를 작성할 때, 자바스크립트는 함수의 용도가 `alert()`과 같이 호출될지 혹은 `new Person()`과 같이 생성자로서 사용될지 추측할 수 없습니다. `Person`과 같은 함수에 `new`를 명시하는 것을 깜빡한다면 오동작이 발생합니다.

Class 문법은 "이것은 단순 함수가 아닙니다. 이것은 class이며 생성자를 가지고 있습니다."라고 말할수 있게 해줍니다. 만약 호출할 때 `new`를 깜빡한다면, 자바스크립트가 에러를 발생시킵니다.

```typescript
let fred = new Person("Fred");
// ✅  Preson이 함수일 때: 정상 작동
// ✅  Person이 클래스일 떄: 정상 작동

let george = Person("George"); // `new` 누락
// 😳 Person이 생성자를 표방하는 함수일 때: 오작동
// 🔴 Person이 클래스일 때: 즉시 실패
```

이 같은 동작은 `this.name`이 `gorge.name` 대신에 `window.name`으로 취금되는 것과 같은 버그를 기다리지 않고 실수를 보다 빨리 인지할 수 있도록 도와줍니다.

하지만, 이것은 React가 클래스를 호출하기 전에 `new`를 붙여야 한다는 것을 의미합니다. 자바스크립트가 버그로 취급하기 때문에, 일반 함수와 같은 방식으로 호출할 수 없습니다.

```tsx
class Counter extends React.Component {
  render() {
    return <p>Hello</p>;
  }
}

// 🔴 React는 이렇게 동작할 수 없습니다:
const instance = Counter(props);
```

여기서 문제가 발생합니다.

---

React가 이 문제를 어떻게 푸는지 알기 전에, 대부분의 사람들은 오래된 브라우저를 위해 Babel과 같은 컴파일러로 class와 같은 최신 기능을 컴파일하여 React를 사용한다는 사실을 기억해야 합니다. 따라서 우리는 컴파일러를 고려해서 디자인해야 합니다.

Babel의 초기 버전에서, class는 `new` 없이 호출할 수 있었습니다. 하지만, 이것은 일부 코드가 추가로 생성되는 것으로 수정됐습니디.

```typescript
function Person(name) {
  // Bebel 출력을 단순화 시켰습니다.
  if (!(this instanceof Person)) {
    throw new TypeError("Cannot call a class as a function");
  }
  // 작성한 코드:
  this.name = name;
}

new Person("Fred"); // ✅ 정상
Person("George"); // 🔴 class를 함수처럼 호출할 수 없습니다.
```

번들에서 위와 같은 코드를 찾아볼 수 있습니다. 이게 `_classCallCheck` 함수의 전부입니다. ("loose mode"를 선택해 체크하지 않는다면 번들 사이즈를 줄일 수 있으나, 실제 네이티브 클래스로 최종 변환이 복잡해질 수 있습니다.)

---

이제 무엇가를 호출하는데 있어 `new`의 존재 여부가 어떤 차이가 있는지 대략적으로 이해해야 합니다.

|            | `new Person()`                        | `Person()`                                  |
| ---------- | ------------------------------------- | ------------------------------------------- |
| `class`    | ✅ `this`는 `Person` 인스턴스 입니다. | 🔴 `TypeError`                              |
| `function` | ✅ `this`는 `Person` 인스턴스 입니다. | 🔴 `this`는 `window` 혹은 `undefied`입니다. |

이것이 React가 컴포넌트를 정확하게 호출해는 것이 중요한 이유입니다. 컴포넌트가 클래스로 정의됐다면, React는 호출 시 `new` 연산자를 사용해야합니다.

그렇다면 React는 무엇이 클래스인지 아닌지 판별할 수 있을까요?

쉽지는 않습니다! [JavaScript의 함수에서 클래스를 구분](https://stackoverflow.com/questions/29093396/how-do-you-check-the-difference-between-an-ecmascript-6-class-and-function)할 수 있다고 할지라도, Babel과 같은 도구로 처리된 클래스에게는 유효하지 않습니다. 브라우저에서는 모든게 일반 함수일 뿐입니다. React에게는 안좋은 소식이죠.

---

그렇다면 만약 React가 모든 호출에서 `new`를 사용하면 되지 않을까요? 불행히도. 항상 통하지는 않습니다.

일반 함수는 `new`와 함께 호출되면 `this`에 해당하는 객체 인스턴스를 반환합니다. 이건 생성자 함수로서 작성된 함수에게는 옳은 방향이지만(위에 작성된 우리의 `Person`과 같이), 함수형 컴포넌트에서는 오작동이 됩니다.

```tsx
function Greeting() {
  // 여기서 `this`가 어떤 종류의 인스턴스가 되는 것을 기대하지 않습니다.
  return <p>Hello</p>;
}
```

이건 그나마 나은 편입니다. 이 아이디어가 묵삻될 수 밖에 없는 두가지 이유가 더 있습니다.

---

항상 `new`를 사용하는 것이 제대로 동작하지 않는 첫 번째 이유는 네이티브 화살표 함수(Babel로 컴파일 된 것이 아닌)가 `new`와 함께 사용되면 에러를 발생시키기 때문입니다.

```typescript
const Greeting = () => <p>Hello</p>;
new Greeting(); // 🔴 Greeting is not a constructor
```

이런 동작은 화살표 함수의 디자인과 깊은 관련이 있습니다. 화살표 함수의 주요 특징중 하나는 스스로의 `this` 값을 갖지 않는다는 겁니다. 대신 상위 스코프에서 가장 가까운 함수의 `this`를 그대로 참조합니다.

```tsx
class Friends extends React.Component {
  render() {
    const friends = this.props.friends;
    return friends.map((friend) => (
      <Friend
        // `this`는 `rener` 메서드의 `this`를 참조합니다.
        size={this.props.size}
        name={friend.name}
        key={friend.id}
      />
    ));
  }
}
```

다시 말해 화살표 함수는 스스로의 `this`를 갖지 않습니다. 이건 화살표 함수가 생성자로서 쓸모가 전혀 없다는 말입니다!

```typescript
const Person = (name) => {
  // 🔴 이건 말이 안됩니다!
  this.name = name;
};
```

게다가, Javascript는 화살표 함수를 `new`와 함께 사용하는 것을 허용하지 않습니다. 만약 했다면, 어떤 이유에서든지 잘 못 사용하고 있는 것이고, 가능한 빨리 알아차리는 것이 중요합니다. 이건 Javascript가 `new` 없이 클래스를 사용하지 못하게 하는 것과 유사합니다.

이 훌륭한 동작은 우리 계획을 방해합니다. React는 화살표 함수가 깨지기 때문에 단순히 모든 함수에 `new`를 붙일 수 없습니다! 우린 화살표 함수가 `prototype`을 갖지 않는다는 점을 이용해서, 화살표 함수를 걸러내어 `new`의 사용을 회피할 수 있습니다.

```typescript
(() => {}).prototype; // undefined
function () {}.prototype; // {constructor: f}
```

하지만 이 방식은 Babel로 컴파일된 함수에서는 [통하지 않습니다.](https://github.com/facebook/react/issues/4599#issuecomment-136562930) 이 문제점은 크지 않을 수 있지만, 이런 접근 방식을 불가능하게 하는 한 가지 이유가 더 있습니다.

> [Javascript의 Arrow Function에 대해 좀 더 알아보기](https://velog.io/@hangem422/js-es6-function#3-%ED%99%94%EC%82%B4%ED%91%9C-%ED%95%A8%EC%88%98)

---

함상 `new`를 사용할 수 없는 다른 이유는 React가 문자열 혹은 다른 원시 타입을 반환하는 컴포넌트를 지원하는데 방해되기 때문입니다.

```tsx
function Greeting() {
  return "Hello";
}

Greeting(); // ✅ 'Hello'
new Greeting(); // 😳 Greeting {}
```

또 다시 [`new` 연산자](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/new) 디자인의 단점과 관련이 있습니다. 앞서 봤듯이, `new`는 Javascript 엔진에게 개게를 생성하도록 명령하며, 함수 내부의 `this` 객체를 만들고, `new` 연산의 결과로 이 객체를 반환합니다.

그러나, Javascript는 `new`로 호출함 함수의 반환값을 오버라이드 하여 다른 객체를 반환하는 것을 허용합니다. 아마 이건 pooling과 같이 인스턴스를 재사용하는 패턴 사용을 고려한 것이 아닐까 생각합니다.

```typescript
// 지연 생성
var zeroVector = null;

function Vector(x, y) {
  if (x === 0 && y === 0) {
    if (zeroVector !== null) {
      // 동일한 인스턴스를 재사용합니다.
      return zeroVector;
    }
    zeroVector = this;
  }
  this.x = x;
  this.y = y;
}

var a = new Vector(1, 1);
var b = new Vector(0, 0);
var c = new Vector(0, 0); // 😲 b === c
```

그러나, `new`는 객체가 아닌 값을 반환하는 것을 철저하게 무시합니다. 만약 string이나 number를 반환하면, 어떤 것도 반환하지 않은 것과 동일합니다.

```typescript
function Answer() {
  return 42;
}

Answer(); // ✅ 42
new Answer(); // 😳 Answer {}
```

`new`와 함께 호출된 함수로부터 원시 타입의 반환값을 받을 수 있는 방법은 존재하지 않습니다(string 혹은 number와 같은). 그렇기 떄문에 만약 React가 항상 `new`를 사용한다면, 문자열을 반환하는 컴포넌트를 추가할 수 없습니다.

이건 받아들일 수 없기 떄문에 다른 방법을 찾아야 합니다.

---

지금까지 무엇을 깨달으셨나요? React는 `new`를 사용하여 클래스(Babel 결과를 포함한)를 호출해야 하며, 일반 함수 혹은 화살표 함수(Babel 결과를 표함한)를 `new` 없이 호출할 수도 있어야 합니다. 그리고 이들을 분류할 수 없습니다.

일반적인 문제를 해결할 수 없다면, 좀더 세부적인 문제를 풀어볼 수는 없을까요?

클래스 컴포넌트를 정의할 때, `this.setState()`와 같은 빌트인 메서드를 사용하기 위해 `React.Component`를 상속하고자 할 것 입니다. 모든 클래스를 감지하는 것 대신에 `React.Component`의 자손을 감지할 수 있을까요?

스포일러: 실제로 React는 이렇게 동작합니다.

---

아마도 `Greeting`이 React component 클래스인지는 `Greeting.prototype instanceof React.Component`로 확인할 수 있을겁니다.

> 이후에는 어떻게 위와 같은 검증 방식이 가능하지에 대해 Javascritp 프로토타입 체인에 대한 설명이 이어집니다.
> 개인적으로 Javascript 포로토타입 체인에대해 이해하고 있기 때문에 해당 내용의 번역을 생략합니다.
> 대신 제가 Javascript 포로토타입을 공부하며 적은 포트팅으로 대체합니다.
> [Javascript의 Prototype에 대해 좀 더 알아보기](https://velog.io/@hangem422/js-prototype)

---

`instanceof` 솔류션의 한가지 주의 사항은 페이지에 복수개의 React 복사본이 존재하고, 검사하고자 하는 컴포넌트와 다른 React 복사본의 `React.Component`로 상속을 채크하면 제대로 동작하지 않는다는 것입니다. 단일 프로젝트에서 여러개의 React 복사본을 섞어 사용하는 것은 몇가지 이유로 좋지 않지만, 옛날부터 우리는 가능한 이슈를 피하려 노력해 왔습니다. (Hooks를 사용하면 중복 제거를 [강제해야 할겁니다.](https://github.com/facebook/react/issues/13991))

또 하나의 가능성 있는 휴리스틱은 프로토타입에 `render` 메서드가 있는지 확인하는 것입니다. 그러나, 그 때 당시에 컴포넌트 API가 어떻게 발전할지 명확하지 않았습니다. 모든 검사는 비용이 들어가므로, 우린 한가지 이상을 검사하고싶지 않았습니다. 만약 클래스 프로퍼티 문법과 동일하게 `render` 메서드가 인스턴스 메서드로 정의될 경우 이 벙법은 유효하지 않습니다.

그 대신에, React는 베이스 컴포넌트에 특별한 플래그를 [추가](https://github.com/facebook/react/pull/4663)했습니다. 리액트는 플래그의 존재 여부를 검사하여 클래스 컴포넌트인지 아닌지를 판단합니다.

원래는 `React.Component` 클래스 내부에 플래그가 존재했습니다.

```tsx
// React 내부
class Component {}
Component.isReactClass = {};

// 다음과 같은 방식으로 검사할 수 있습니다.
class Greeting extends Component {}
console.log(Greeting.isReactClass); // ✅ Yes
```

그러나, 우리가 목표했던 몇가지 클래스 구현에서 static 프로퍼티들을 복사하지 않았고(혹은 비표준 `__proto__`를 설정하거나), 플래그가 누락됐습니다.

이것은 React가 플래그를 `React.Component.prototyp`로 [이동](https://github.com/facebook/react/pull/5021)한 이유입니다.

```tsx
// React 내부
class Component {}
Component.prototype.isReactComponent = {};

// 다음과 같은 방식으로 검사할 수 있습니다.
class Greeting extends Component {}
console.log(Greeting.prototype.isReactComponent); // ✅ Yes
```

그리고 이것은 이름의 의미가 이것의 모든 것이다.

왜 단순 불리언 값이 아닌 객체인지 궁금할 겁니다. 실제로 중요한 것은 아니지만 Jest 초기 버전(Jest가 Good^TM^이기 이전에)은 오토 모킹이 디폴트로 켜져 있었습니다. 생성된 모의 객체는 원시 프로퍼티를 생략했으며, [검사를 중단](https://github.com/facebook/react/pull/4663#issuecomment-136533373)했습니다. 고마워요, Jest.

`isReactComponent` 검사는 오늘날에도 [React에서 사용](https://github.com/facebook/react/blob/769b1f270e1251d9dbdce0fcbd9e92e502d059b8/packages/react-reconciler/src/ReactFiber.js#L297-L300)됩니다.

만약 `React.Component`를 상속받지 않으면, React는 `isReactComponent` 프로퍼티를 검사하지 않을 것이며, 클래스 컴포넌트로 취급받지 못할 겁니다. 이제 왜 `Cannot call a class as a function` 에러의 답변중에서 [가장 많은 추천을 받은 답변](https://stackoverflow.com/a/42680526/458193)이 `extends React.Component`를 추가하라는 것인지 알 수 있을겁니다. 마지막으로, `prototype.render`가 있지만 `prototype.isReactComponent`가 없다는 것에 대한 [경고가 추가됐습니다.](https://github.com/facebook/react/pull/111680)

---

이 이야기가 약간 낚시가 아니냐고 생각할 수 있습니다. 실제 해결 방법은 매우 간단하지만, 왜 React가 이런 이런 솔류션을 선택했고 대안이 어떤 것들이 있었는지 설명하기 위해 크게 돌아갔습니다.

라이브러리 API들과 관련해서 자주 있었던 내 경험상, API를 간단하게 사용하려면 언어의 의미 체계(몇몇 언어에 대해서는 가능하면 향후 계획 까지 포함해서), 런타임 성능, ergonomics with and without compile-time steps, 생태계와 패키징 솔류션의 상태, early warnings 등등을 고려해야 합니다. 최종 결론은 언제나 가장 우아한건 아니지만, 항상 실용적입니다.

만약 최종 API가 성공적이면, 유저는 과정에 대해서 신경 쓸 필요가 없습니다. 대신 앱을 만드는데 집중할 수 있습니다.

하지만 궁금하다면… 어떻게 작동하는지 아는 것이 좋습니다.
