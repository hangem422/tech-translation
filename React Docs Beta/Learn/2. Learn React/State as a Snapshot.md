# 스냅샷(Snapshot)같은 상태(State)

> 원문: [React Docs Beta > Learn > Learn React > State as a Snapshot](https://beta.reactjs.org/learn/state-as-a-snapshot)

상태(State) 변수는 일고 쓸 수 있다는 점에서 일반적인 자바스크립트 변수와 유사해 보입니다. 하지만, 상태(State)의 동작은 스냅샷과 더 유사합니다. 상태(State)를 설정하는 것은 기존에 가지고있던 상태를 변경하는 것이 아니라, 리랜더를 발생시키는 것 입니다.

> 이런 것을 배워요
>
> - 상태(State)를 설정하는 것이 어떻게 리렌더를 일으키는가
> - 언제 그리고 어떻게 상태(State)가 업데이트 되는가?
> - 상태(State)는 왜 설정하자마자 업데이트되지 않는가?
> - 이벤트 핸들러는 상태(State)의 "snapshot"에 어떻게 접근하는가?

## 상태(State) 설정은 리렌더를 일으킨다

유저 인터페이스가 클릭과 같은 유저 이벤트에 반응하여 즉시 변한다 생각할 수 있습니다. React는 이런 멘탈 모델과 조금 다르게 동작합니다. 이전 페이지에서, [상태(State) 설정이 React로부터 리렌더를 요청](https://beta.reactjs.org/learn/render-and-commit#step-1-trigger-a-render)하는 것을 보셨습니다. 인터페이스가 이벤트에 반응하기 위해서는, 상태(State)를 업데이트해야 한다는 것을 의미합니다.

이 예제에서, "send"를 누르면 `setIsSent(true)`가 React에게 UI를 리렌더해야 한다고 말합니다.

```tsx
import { useState } from "react";

export default function Form() {
  const [isSent, setIsSent] = useState<boolean>(false);
  const [message, setMessage] = useState<string>("Hi!");
  if (isSent) {
    return <h1>Your message is on its way!</h1>;
  }
  return (
    <form
      onSubmit={(e) => {
        e.preventDefault();
        setIsSent(true);
        sendMessage(message);
      }}
    >
      <textarea placeholder="Message" value={message} onChange={(e) => setMessage(e.target.value)} />
      <button type="submit">Send</button>
    </form>
  );
}

function sendMessage(message) {
  // ...
}
```

여기 버튼을 클릭하면 일어나는 일들이 있습니다.

1. `onSubmit` 이벤트 핸들러가 실행됩니다.
2. `setIsSent(true)`가 `isSent`를 `true`로 설정하고 새로운 렌더가 대기열에 추가됩니다.
3. React가 새로운 `isSent` 값을 반영하여 컴포넌트를 리렌더합니다.

상태와 렌더링 사이의 관계를 좀 더 자세히 알아봅시다.

## Rendering은 당시의 스냅샷을 촬영합니다

렌더링은 React가 컴포넌트 함수를 호출하는 것을 의미합니다. 함수에서 반환된 JSX는 당시의 UI 스냅샷과 같습니다. props, 이벤트 핸들러, 그리고 로컬 변수들은 **렌더 당시의 상태(State)를 이용해** 계산됩니다.

영화 프레임 혹은 사진과 다르게, 반환된 UI "스냅샷"은 반응형입니다. 입력에 특정한 반응이 일어나는 이벤트 핸들러 같은 로직을 포함합니다. React는 스냅샷과 화면을 일치시키고나서 이벤트 핸들러를 연결합니다. 결과적으로 버튼을 누르는 것은 JSX의 클릭 핸들러를 호출하는 것입니다.

React가 컴포넌트를 리렌더 한다면:

1. React는 컴포넌트 함수를 다시 호출합니다.
2. 함수가 새로운 JSX 스냅샷을 반환합니다.
3. 반환된 스냅샷과 일치하게 화면을 업데이트합니다.

컴포넌트의 메모리로서, 상태(State)는 함수가 결과를 반환한 후 사라지는 일반 변수들과 다릅니다. 컴포넌트가 적제되어 있다면 상태(State)는 컴포넌트 함수 밖 React 내부에서 스스로 살아남습니다. 리엑트는 컴포넌트를 호출할 때, 특정 렌더에 대한 상태(State)의 스냅샷을 제공합니다. 컴포넌트는 새로운 props와 이벤트 핸들러를 JSX에 담은 UI의 스냅샷을 반환하며, 이 모든 것은 **해당 렌더의 상태(State) 값을 가지고 계상됩니다.**

여기 동작 방식에대한 작은 예제가 있습니다. 예제에서, "+3" 버튼을 클릭하면 `setNumber(number + 1)`을 세번 호출하기 때문에 카운트가 3만큼 올라갈 것이라 예상할 수도 있습니다.

실제 "+3" 버튼을 클릭하면 무슨 일이 일어나는지 확인해보세요.

```tsx
import { useState } from "react";

export default function Counter() {
  const [number, setNumber] = useState<number>(0);

  return (
    <>
      <h1>{number}</h1>
      <button
        onClick={() => {
          setNumber(number + 1);
          setNumber(number + 1);
          setNumber(number + 1);
        }}
      >
        +3
      </button>
    </>
  );
}
```

클릭당 `number`가 1밖에 증가하지 않습니다.

**상태(State) 설정은 오직 다음 렌더의 값만을 변경합니다.** 첫 렌더에서, `number`는 `0`입니다. 이것이 렌더의 `onClick` 핸들러에서, `setNumber(number + 1)`가 호출됐음에도 `number`가 계속 `0`인 이유입니다.

```tsx
<button
  onClick={() => {
    setNumber(number + 1);
    setNumber(number + 1);
    setNumber(number + 1);
  }}
>
  +3
</button>
```

이 버튼의 클릭 핸들러가 React에게 할 일을 만해주는 것은 다음과 같습니다.

1. `setNumber(number + 1)`: `number`는 `0`이며 `setNumber(0 + 1)`가 됩니다.
   - React는 다음 렌더에서 `number`를 `1`로 바꿀 준비를 합니다.
2. `setNumber(number + 1)`: `number`는 `0`이며 `setNumber(0 + 1)`가 됩니다.
   - React는 다음 렌더에서 `number`를 `1`로 바꿀 준비를 합니다.
3. `setNumber(number + 1)`: `number`는 `0`이며 `setNumber(0 + 1)`가 됩니다.
   - React는 다음 렌더에서 `number`를 `1`로 바꿀 준비를 합니다.

`setNumber(number + 1)`를 세번이나 호출했음에도 불구하고, 이번 렌더의 이벤트 핸들러에서 `number`는 항상 `0`입니다. 그렇기 때문에 상태(State)를 1로 세번을 설정하게 됩니다. 이벤트 핸들러의 동작이 끝나고, React가 컴포넌트의 `number`가 `3`이 아닌 `1`로서 리렌더 하는 이유입니다.

상태(State) 변수를 코드 내의 값으로 치환하여 시각화해 볼 수 있습니다. `number` 상태 값이 `0`인 이번 렌더에서, 이벤트 핸드러는 다음과 같습니다.

```tsx
<button
  onClick={() => {
    setNumber(0 + 1);
    setNumber(0 + 1);
    setNumber(0 + 1);
  }}
>
  +3
</button>
```

다음 렌더에서, `number`는 `1`이며, 이벤트 핸들러는 다음과 같습니다.

```tsx
<button
  onClick={() => {
    setNumber(1 + 1);
    setNumber(1 + 1);
    setNumber(1 + 1);
  }}
>
  +3
</button>
```

이건이 버튼을 추가로 클릭했을 때 카운터가 `2`가 되고, `3`이 되는 이유입니다.

## 시간 위의 상태

다음 버튼을 클릭하면 무엇이 alert될 지 추측해보세요.

```tsx
import { useState } from "react";

export default function Counter() {
  const [number, setNumber] = useState<number>(0);

  return (
    <>
      <h1>{number}</h1>
      <button
        onClick={() => {
          setNumber(number + 5);
          alert(number);
        }}
      >
        +5
      </button>
    </>
  );
}
```

이전의 대체 방법을 사용하면, alert이 "0"을 출력할 것이라 추측할 수 있습니다.

```ts
setNumber(0 + 5);
alert(0);
```

하지만 alter에 타이머를 두어, 리랜더 후에 실행되도록 하면 어떻게 될까요? "0"일까요 "5"일까요? 맞춰보시죠!

```tsx
import { useState } from "react";

export default function Counter() {
  const [number, setNumber] = useState<number>(0);

  return (
    <>
      <h1>{number}</h1>
      <button
        onClick={() => {
          setNumber(number + 5);
          setTimeout(() => {
            alert(number);
          }, 3000);
        }}
      >
        +5
      </button>
    </>
  );
}
```

놀라셨나요? 만약 데체 방법을 사용했단면, alert으로 넘어가는 상태(State)의 "스냅샷"을 생각할 수 있었을 겁니다.

```ts
setNumber(0 + 5);
setTimeout(() => {
  alert(0);
}, 3000);
```

React에 저장된 상태(State)가 alert이 실행될 때 변했을 수 있습니다. 하지만, 유저가 인터렉션을 수행했을 당시 상태(State)의 스냅샷을 사용하도록 스케줄 돼 있습니다.

이벤트 핸들러 코드가 비동기라 할지라도, **상태(State) 변수는 렌더 중간에 절대 변하지 않습니다.** 렌더 중의 `onClick` 내부에서, `setNumber(number + 5)`가 호출된다 할지라도 `number`의 값은 항상 `0`으로 유지됩니다. React가 컴포넌트를 호출하여 UI의 "스냅샷"을 만들 때 값이 "고정"됩니다.

여기 이벤트 핸들러의 타이밍 실수를 줄이기 위한 방법 예시가 있습니다. 5초 후에 메시지를 보내는 Form입니다. 시나리오를 상상해보세요.

1. "Send" 버튼을 눌러 Alice에게 "Hello"를 보냅니다.
2. 5초의 지연이 끝나기 전에, "To" 필드를 "Bob"으로 변경합니다.

`alert`에 무엇이 출력될 것을 예상하시나요? “You said Hello to Alice”일까요, “You said Hello to Bob”일까요?

```tsx
import { useState, type MouseEvent } from "react";

export default function Form() {
  const [to, setTo] = useState<string>("Alice");
  const [message, setMessage] = useState<string>("Hello");

  function handleSubmit(e: MouseEvent<HTMLButtonElement>) {
    e.preventDefault();
    setTimeout(() => {
      alert(`You said ${message} to ${to}`);
    }, 5000);
  }

  return (
    <form onSubmit={handleSubmit}>
      <label>
        To:{" "}
        <select value={to} onChange={(e) => setTo(e.target.value)}>
          <option value="Alice">Alice</option>
          <option value="Bob">Bob</option>
        </select>
      </label>
      <textarea placeholder="Message" value={message} onChange={(e) => setMessage(e.target.value)} />
      <button type="submit">Send</button>
    </form>
  );
}
```

**React는 렌더의 이벤트 핸들러 내부의 상태(State) 값을 "고정" 합니다. 코드가 실행중에 상태가 변경될 걱정을 하지 않아도 됩니다.**

하지만 리렌더 전에 최신 상태(State)를 읽고 싶다면 어떻게 해야할까요? 다음 페이지에서 다룰 [상태 업데이트 함수](https://beta.reactjs.org/learn/queueing-a-series-of-state-updates)를 사용해야 합니다.

## 요약

- 상태(State) 설정은 리랜더를 요청한다.
- React는 컴포넌트가 적제 됐을 때, 상태를 컴포넌트 밖에 저장한다.
- `useState`를 사용하면, React는 해당 렌더의 상태의 스냅샷을 제공합니다.
- 변수와 이벤트 핸들러는 리렌더에서 살아남지 못합니다. 모든 렌더에서 자신만의 이벤트 핸들러를 갖습니다.
- 모든 리렌더(와 내부에 있는 함수)는 언제나 React가 제공하는 해당 렌더의 상태의 스냅샷을 바라봅니다.
- 이벤트 핸들러 내부의 상태를 렌더된 JSX를 상상하는 방법과 유사하게 대체하여 생각할 수 있습니다.
- 과거에 만들어진 이벤트 핸들러는 자신이 만들어 질 때의 상태 값을 가지고 있습니다.
