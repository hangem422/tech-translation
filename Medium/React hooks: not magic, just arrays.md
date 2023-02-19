# React Hooks: 마술이 아니라 그저 배열일 뿐이다

> Rudi Yardley, [React hooks: not magic, just arrays](https://medium.com/@ryardley/react-hooks-not-magic-just-arrays-cd4f1857236e), Nov 1, 2018

> 다이어그램을 이용해 제안서에 있는 규칙들을 풀어봅니다.

저는 새로운 Hooks API에 큰 팬입니다. 그러나, Hooks는 사용하기 위해 필요한 [이상한 제약사항들](https://reactjs.org/docs/hooks-rules.html)을 가지고 있습니다. 이런 규칙들의 이유를 이해하기 위해 고군분투하는 이들을 위해, 새 API를 사용하는 것을 어떻게 생각해야 하는지를 위한 모델을 제안하고자 합니다.

## Hooks의 동작 방식 살펴보기

몇몇 사람들이 새로운 Hooks API 초안에 담긴 "마법"에 대해 고군분투하고 있다 들었습니다. 그래서 나는 구문 제언서(Syntax Proposal)가 어떻게 동작하는지 최소한 표면적으로라도 살펴보고자 시도했습니다.

### Hooks의 규칙들

Hooks을 사용하기 위해 React 코어 팀이 요구하는 사용 규칙은 크게 두 가지가 있으며, [Hooks 제언 문서](https://reactjs.org/docs/hooks-rules.html)에 명시돼 있습니다.

- **반복문, 조건문, 중첩 함수 내부에서 Hooks를 호출하지 마세요.**
- **React Function에서만 Hooks를 호출하세요.**

후자는 자명하다고 생각합니다. 함수형 컴포넌트에 동작을 붙이기 위해서는 해당 동작을 컴포넌트와 연결할 수 있어야 합니다.

그러나 전자는 이런 방식으로 API를 사용해 프로그래밍하는 것이 부자연스러워 보일 수 있기 때문에 혼란스러울 수 있다고 생각합니다. 그리고 이것이 오늘 제가 탐구하고자 하는 주제입니다.

### Hooks에서의 State 관리는 모두 배열과 관련이 있습니다.

명확한 멘탈 모델(Mental Model)을 위해, Hooks API의 구현이 어떻게 생겼는지 간략하게 살펴 봅시다.

_이것은 추측이며 단지 당신이 Hooks를 어떻게 생각해야 하는지 보여주기 위한 API를 구현하는 방법중 하나라는 것에 유의하세요. 실제 API가 내부적으로 동작하는 방법이 아닙니다._

### "useState()"를 어떻게 구현할까?

State Hook 구현이 대략적으로 어떻게 동작하는지 보여주는 예시를 살펴봅시다.

먼저 컴포넌트를 보겠습니다.

```tsx
function RenderFunctionComponent() {
  const [firstName, setFirstName] = useState("Rudi");
  const [lastName, setLastName] = useState("Yardley");

  return <Button onClick={() => setFirstName("Fred")}>Fred</Button>;
}
```

Hooks API의 아이디어는 Hook 함수가 반환하는 배열의 두 번째 요소로서 setter 함수를 사용할 수 있고, setter가 Hook에 의해 관리되는 state를 컨트롤 한다는 것입니다.

## 그래서 React는 이것으로 무엇을 할까?

이것이 React 안에서 내부적으로 대략 어떻게 동작하는지 이야기 해봅시다. 다음 과정은 특정 컴포넌트를 렌더링하기 위해 실행 컨텍스트에서 발생합니다. 따라서 여기서 저장되는 데이터는 렌더링되는 컴포넌트의 한 단계 밖에 있습니다. 이 state는 다른 컴포넌트와 공유되지 않지만, 특정 컴포넌트의 후속 렌더링(Subsequent Render)에서 접근이 가능한 스코프(scope)에서 유지됩니다.

### 1) 초기화

"setter"와 "state" 두 개의 배열을 생성합니다.

cursor를 0으로 맞춥니다.

![초기화: 두개의 빈 배열 생성, 커서는 0](https://miro.medium.com/v2/resize:fit:1280/format:webp/1*LAZDuAEm7nbcx0vWVKJJ2w.png)

### 2) 첫번째 렌더

컴포넌트 함수를 처음을 실행합니다.

첫 실행에서, 각 `useState()`는 setter 함수(cursor 위치가 바이딩 된)를 `setters` 배열에 푸쉬하고, `state` 배열에 어떤 state를 푸쉬합니다.

![첫번째 렌더: 배열에 아이템이 기록됨과 동시에 cursor가 증가한다.](https://miro.medium.com/v2/resize:fit:1260/format:webp/1*8TpWnrL-Jqh7PymLWKXbWg.png)

### 3) 후속 렌더(Subsequent Render)

후속 렌더마다 cursor는 초기화되고, 해당하는 값들은 각 배열로부터 가져와집니다.

![후속 렌더: 배열로부터 아이템이 읽어짐과 동시에 cursor가 증가한다.](https://miro.medium.com/v2/resize:fit:1254/format:webp/1*qtwvPWj-K3PkLQ6SzE2u8w.png)

### 4) 이벤트 핸들링

각 setter는 자신의 cursor 위치에 대한 참조를 가지고 있기 때문에, 어떤 `setter` 호출이 발생하면 state 배열에서 해당 위치의 state 값을 수정할 수 있습니다.

![Setter들은 그들의 index를 기억하고 이에 따른 메모리를 수정합니다.](https://miro.medium.com/v2/resize:fit:1260/format:webp/1*3L8YJnn5eV5ev1FuN6rKSQ.png)

### 나이브한 구현

다음은 구현을 보여주는 나이브한 코드 예시입니다.

_주의: 이것은 Hooks의 전체 동작 방식을 보여주지는 않지만 단일 컴포넌트에서 Hook이 어떻게 동작하는지 생각해보는 좋은 방향성을 제시합니다. 이건 모듈 레벨의 변수 같은 것들을 사용한 이유입니다._

```tsx
let state: string[] = [];
let setters: Array<(newVal: string) => void> = [];
let firstRun: boolean = true;
let cursor: number = 0;

function createSetter(cursor: number) {
  return function setterWithCursor(newVal: string) {
    state[cursor] = newVal;
  };
}

//  useState helper의 유사 코드입니다
export function useState(initVal) {
  if (firstRun) {
    state.push(initVal);
    setters.push(createSetter(cursor));
    firstRun = false;
  }

  const setter = setters[cursor];
  const value = state[cursor];

  cursor++;
  return [value, setter];
}

// Hooks를 사용하는 컴포넌트 코드
function RenderFunctionComponent() {
  const [firstName, setFirstName] = useState("Rudi"); // cursor: 0
  const [lastName, setLastName] = useState("Yardley"); // cursor: 1

  return (
    <div>
      <Button onClick={() => setFirstName("Richard")}>Richard</Button>
      <Button onClick={() => setFirstName("Fred")}>Fred</Button>
    </div>
  );
}

// 이것은 일종의 React 렌더링 사이클을 시뮬레이션 하는 것 입니다.
function MyComponent() {
  cursor = 0; // resetting the cursor
  return <RenderFunctionComponent />; // render
}

console.log(state); // 렌더 이전: []
MyComponent();
console.log(state); // 첫번째 렌더: ['Rudi', 'Yardley']
MyComponent();
console.log(state); // 뒤이어 일어나는 렌더: ['Rudi', 'Yardley']

// 'Fred' 버튼 클릭

console.log(state); // 클릭 이후: ['Fred', 'Yardley']
```

## 순서가 중요한 이유

만약 외부 요인 혹은 컴포넌트 상태로인해 렌더 사이클에서 Hooks의 순서를 변경하면 어떻게 될까요?

React 팀이 하지 말라고 한 것을 해봅시다.

```tsx
let firstRender = true;

function RenderFunctionComponent() {
  let initName;

  if (firstRender) {
    [initName] = useState("Rudi");
    firstRender = false;
  }
  const [firstName, setFirstName] = useState(initName);
  const [lastName, setLastName] = useState("Yardley");

  return <Button onClick={() => setFirstName("Fred")}>Fred</Button>;
}
```

여기 조건적으로 호출되는 `useState`가 있습니다. 이것이 시스템에 만드는 대파괴를 살펴봅시다.

### 나쁜 컴포넌트의 첫 렌더

![다음 렌더에서 사라질 "나쁜" Hook의 렌더링](https://miro.medium.com/v2/resize:fit:1270/format:webp/1*C4IA_Y7v6eoptZTBspRszQ.png)

이 시점에서 `firstName`와 `lastName` 변수는 정확한 데이터를 가지고 있지만, 두번째 렌더부터 어떤 일이 일어나는지 살펴봅시다.

### 나쁜 컴포넌트의 두 번째 렌더

![렌더 중간에 후크를 제거하면 오류가 발생합니다.](https://miro.medium.com/v2/resize:fit:1274/format:webp/1*aK7jIm6oOeHJqgWnNXt8Ig.png)

이제 `firstName`와 `lastName`는 state 저장소와 엇갈리면서 둘 다 `Rudi`가 할당됩니다. 이건 명백히 잘못됐고 동작하지 않지만 Hooks의 규칙이 존재하는 이유를 있는 그대로 보여줍니다.

> 지키지 않으면 데이터 불일치가 발생하기 때문에 React 팀은 사용 규칙을 규정하고 있습니다.

### Hooks가 배열들의 집합을 조작한다고 생각하면 규칙을 깨지 않을 것 입니다.

이제 왜 "use" Hooks를 조건문이나 반복문에서 사용하면 안되는지 명확해 졌습니다. 배열들의 집합의 커서 포인터를 다루기 때문에, 랜더 중 호출 순서를 바꾼다면, 커서와 데이터가 일치하지 않게 되고 "use" 호출은 정확한 데이터 혹은 핸들러를 가르키지 못합니다.

따라서 Hooks를 일관된 커서가 필요한 배열의 집합으로 비즈니스(business)를 관리하는 것으로 생각하는 것이 요령입니다. 이렇게만 생각하면 모든게 이해됩니다.

## 결말

제가 새로운 Hooks API 내부에서 어떤 일이 일어나는지에 대해 어떻게 생각해야 하는지 명확한 멘탈 모델을 제시했기를 바랍니다. 여기서 기억해야 할 진정한 가치는 관심사를 함께 묶을 수 있기 때문에, 순서에 유의하여 Hooks API를 사용하면 큰 이점을 취할 수 있다는 것입니다.

Hooks는 React 컴포넌트를 위한 효과적인 플러그인 API입니다. 사람들이 이것에 흥분하는데는 이유가 있으며, 이것을 state가 배열의 집합으로서 존재하는 하나의 모델로 생각한다면 사용 규칙을 어기지 않을 수 있습니다.
