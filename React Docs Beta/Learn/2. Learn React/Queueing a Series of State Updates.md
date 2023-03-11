# 상태(State) 업데이트 대기열

> 원문: [React Docs Beta > Learn > Learn React > Queueing a Series of State Updates](https://beta.reactjs.org/learn/queueing-a-series-of-state-updates)

상태(State) 변수를 설정하면 다른 렌더가 대기열에 들어갑니다. 다음 랜더가 대기열에 들어가기 전에 값에대한 다수의 명령을 수행하고 싶을 수 있습니다. React가 상태 업데이트를 묶는 방법에 대해 이해해 보도록 합시다.

> ### 이런 것을 배워요
>
> - "batching"이 어떤 것이고 React가 다수의 상태 업데이트를 위해 이것을 어떻게 사용하는지
> - 동일한 상태 변수를 여러번 연속적으로 수정하는 방법

## React는 업데이트들을 묶는다

“+3” 버튼을 클릭하면 `setNumber(number + 1)`를 세 번 호출하기 때문에 카운터를 세번 증가할 것이라 생각할 수 있습니다.

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

하지만, [각 렌더의 상태(State) 값은 고정돼있으며](https://beta.reactjs.org/learn/state-as-a-snapshot#rendering-takes-a-snapshot-in-time), 첫 렌더의 이벤트 핸들러 속 `number`는 언제나 `0`이기에, 호출한 횟수와 상관 없이 `setNumber(1)`를 호출합니다.

```ts
setNumber(0 + 1);
setNumber(0 + 1);
setNumber(0 + 1);
```

하지만 여기서 논의해볼 또 다른 한가지 사실이 있습니다. **React는 상태를 업데이트하기 전에, 이벤트 핸들러들의 모든 코드가 싱핼될 때까지 기다립니다.** 이것이 모든 `setNumber()`가 호출된 후에 리랜더가 일어나는 이유입니다.

레스토랑에서 웨이터가 주문을 받는 것을 생각해 볼 수 있습니다. 웨이터는 첫 음식만 듣고 부엌으로 뛰어가지 않습니다! 대신에, 주문이 끝날 때 까지 기다리고, 주문에 대한 변경도 받으며, 테이블 내에 다른 손님의 주문 또한 받습니다.

이건 복수의 컴포넌트서 복수개의 상태 변수를 변경할 지라도 너무 많은 리렌더를 하지 않게 해줍니다. 그러나 이건 이벤트 핸들러의 모든 코드가 완료될 떄 까지 UI가 업데이트되지 않는 다는 것을 의미하기도 합니다. **Batching**이라고도 불리는 이런 동작은 React 앱을 빠르게 만들어 줍니다. 또한, 일부의 변수만 업데이트 된 "반쪽짜리" 렌더로 인한 혼란을 필할 수 있게 해줍니다.

React는 별도로 일어나는 클릭과 같이 의도된 이벤트들을 묶지는 않습니다. React는 일반적으로 안전할 떄만 Batching을 수행하므로 안심하셔도 됩니다. 첫 번쨰 버튼이 Form을 비활성화 처리하여, 두 번쨰 클릭으로 다시 제출되지 않게 하는 것과 같은 동작을 보장합니다.

## 다음 렌더 전에 동일한 상태 변수를 여러번 업데이트하기

일반적이지 않은 경우이지만, 다음 렌더 전에 동일한 상태 변수를 여러번 업데이트해야 한다면, `setNumber(number + 1)`와 같이 다음 상태 값을 넘기는 것 대신에, `setNumber(n => n + 1)`와 같이 대기열의 이전 상태 값으로 다음 상태 값을 계산하는 함수를 넘길 수 있습니다. 이건 React에게 단순 값을 변경하는 것이 아니라 "상태 값으로 무언가를 해줘"라고 말하는 방법입니다.

카운트를 증가시키는 것을 해봅시다.

```tsx
import { useState } from "react";

export default function Counter() {
  const [number, setNumber] = useState<number>(0);

  return (
    <>
      <h1>{number}</h1>
      <button
        onClick={() => {
          setNumber((n) => n + 1);
          setNumber((n) => n + 1);
          setNumber((n) => n + 1);
        }}
      >
        +3
      </button>
    </>
  );
}
```

여기 `n => n + 1`는 **업데이트 함수**라고 부릅니다. 상태 Setter에 넘겼을 떄:

1. React는 이 함수가 이벤트 핸들러의 다른 코드가 끝난 후에 실행되도록 대기열에 넣습니다.
2. 다음 렌더가 진행되는 동안, React는 대기열을 퉁해 최종 업데이트 된 상태를 제공합니다.

```ts
setNumber((n) => n + 1);
setNumber((n) => n + 1);
setNumber((n) => n + 1);
```

이벤트 핸들러를 실행하는 동안 React가 위 코드에대해 작동하는 방법은 다음과 같습니다.

1. `setNumber(n => n + 1)`: `n => n + 1`은 함수입니다. React는 이것을 대기열에 추가합니다.
2. `setNumber(n => n + 1)`: `n => n + 1`은 함수입니다. React는 이것을 대기열에 추가합니다.
3. `setNumber(n => n + 1)`: `n => n + 1`은 함수입니다. React는 이것을 대기열에 추가합니다.

다음 렌더에서 `useState`를 호출했을 떄, React는 대기열을 훑습니다. 이전 `number` 상태는 `0`이었고, React 이것을 첫 번쨰 업데이트 함수의 `n` 인자로 넘겨줍니다. 그리고 React는 이전 업데이트 함수의 반환 값을 다음 업데이트 함수의 `n` 인자로 넘기는 것을 이어갑니다.

| 대기열의 업데이트 함수 | `n` | 반환값      |
| ---------------------- | --- | ----------- |
| `n => n + 1`           | `0` | `0 + 1 = 1` |
| `n => n + 1`           | `1` | `1 + 1 = 2` |
| `n => n + 1`           | `2` | `2 + 1 = 3` |

React는 `3`을 최종 결과로서 저장하고 `useState`에서 반환합니다.

위의 예제에서 “+3”에서 값이 정확하게 3으로 증가한 이유입니다.

### 업데이트 후에 이것을 변경하면 어떻게 될까?

한가지 예제를 더 시도해 봅시다.

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
          setNumber((n) => n + 1);
          setNumber(42);
        }}
      >
        Increase the number
      </button>
    </>
  );
}
```

이벤트 핸들러를 실행하는 동안 React가 위 코드에대해 작동하는 방법은 다음과 같습니다.

1. `setNumber(number + 5)`: `number`는 `0`이며, `setNumber(0 + 5)`가 됩니다. React는 “replace with 5”를 대기열에 추가합니다.
2. `setNumber(n => n + 1)`: `n => n + 1`는 업데이트 함수입니다. React는 이 함수를 대기열에 추가합니다.
3. `setNumber(42)`: React는 “replace with 42”를 대기열에 추가합니다.

다음 렌더 동안에 React는 상태 대기열을 훑습니다.

| 대기열의 업데이트 함수 | `n`          | 반환값      |
| ---------------------- | ------------ | ----------- |
| “replace with 5”       | `0` (unused) | `5`         |
| `n => n + 1`           | `5`          | `5 + 1 = 6` |
| “replace with 42”      | `6` (unused) | `42`        |

React는 `42`를 최종 결과로 저장하고 `useState`에서 반환합니다.

`setNumber` 상태 Setter에 전달하는 것에 대해 요약하자면

- **업데이트 함수**(e.g. `n => n + 1`)는 대기열에 추가됩니다.
- **기타 다른 값** (e.g. number `5`) 이전의 대기열을 무시하고 “replace with 5"를 대기열에 추가합니다.

이벤트 핸들러가 완료되고, React는 리랜더를 일이킵니다. 리렌더 동안에, React는 대기열을 실행합니다. 업데이트 함수는 리렌더 동안에 수행되기 떄문에, **순수해야 하며** 결과를 반환해야 합니다. 이 안에서 상태를 수정하거나 다른 사이드 이펙트를 일으키지 마세요. 엄격 모드(Strcit Mode)에서, React는 각 업데이트 함수를 두번 수행하여(두번째 수행 결과는 버립니다) 실수를 찾아내는 것을 도와줍니다.

## 명명 규칙

상태 변수의 첫 글자로 업데이트 함수 인자 이름을 만드는 것이 일반적입니다.

```ts
setEnabled((e) => !e);
setLastName((ln) => ln.reverse());
setFriendCount((fc) => fc * 2);
```

더 상세한 코드를 원할 때, 다른 하나의 규칙은 `setEnabled(enabled => !enabled)`와 같이 상태 변수의 이름을 그대로 반복하거나, `setEnabled(prevEnabled => !prevEnabled)`와 같이 접두사를 붙이는 것 입니다.

## 요약

- 상태를 설정하는 것은 기존 렌더의 값을 변경하는 것이 아니지만, 새로운 렌더를 요청합니다.
- React는 이벤트 핸들러의 동작이 끝난 후 상태 업데이트를 수행합니다. 이것을 Batching이라 부릅니다.
- 한 이벤트에서 어떤 상태를 여러번 업데이트해야 한다면, `setNumber(n => n + 1)`와 같은 업데이트 함수를 사용할 수 있습니다.
