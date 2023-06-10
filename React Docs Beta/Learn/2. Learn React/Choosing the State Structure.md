# 상태 구조 선택하기

> 원문: [React Docs Beta > Learn > Learn React > Choosing the State Structure](https://react.dev/learn/choosing-the-state-structure)

잘 구조화된 상태는 수정과 디버그에 용이한 컴포넌트와, 지속적으로 버그의 원인이 되는 컴포넌트의 차이를 만들어 냅니다. 다음은 상태의 구조를 고민할 때 고려해야 할 팁입니다.

> 이런 것을 배워요
>
> - 싱글 상태 값 사용 vs 멀티 상태 값 사용
> - 상태를 구성할 때 피해야 하는 것
> - 상태 구조와 관련된 일반적인 문제 해결 방법

## 상태 구조화 원칙

상태를 지닌 컴포넌트를 작성할 때, 얼마나 많은 상태 값을 사용하고 어떤 자료 구조를 가질지 선택해야 합니다. 최선의 상태 구조가 아니더라도 정확한 프로그램을 작성할 수 있지만, 좀 더 나은 선택을 위해 몇 가지 원칙이 존재합니다.

1. **관련된 상태를 묶으세요.** 만약 매번 두 개 이상의 상태를 동시에 갱신한다면, 이들을 하나의 상태 변수로 묶는 것을 고려해 보세요.
2. **상태의 모순을 피하세요.** 몇몇 상태의 조각들이 모순되고 불일치하는 방향으로 상태가 구성되면, 실수의 여지를 남기게 됩니다.
3. **중복된 상태를 피하세요.** 렌더링 과정에서 컴포넌트의 prop 혹은 이미 존재하는 상태 변수로 정보를 계산할 수 있다면, 컴포넌트의 상태로 넣지 말아야 합니다.
4. **복제된 상태를 피하세요.** 한 개의 데이터가 복수의 상태 변수로 복제되거나, 객체 내부에 중첩되면 이들의 동기화를 유지하기 어렵습니다. 가능하면 중복을 줄이세요.
5. **깊이 중첩된 상태를 피하세요.** 깊은 계층적 상태는 갱신하기 쉽지 않습니다. 가능하다면, 상태를 평평하게 구조화하는 것을 지향하세요.

이 원칙들의 목적은 실수 없이 상태 업데이를 쉽게 만들기 위함입니다. 중복되고 복제된 데이터를 상태에서 제거하여 모든 상태의 조각들이 동기화를 유지하도록 돕습니다. 이것은 데이터 베이스 엔지니어가 버그를 줄이기 위해 [데이터 베이스 구조를 "정규화"](https://docs.microsoft.com/en-us/office/troubleshoot/access/database-normalization-description)하기 원하는 것과 유사합니다. 알버트 아인슈타인의 말을 빌리자면, **"당신의 상태를 가능한 한 단순하게 하라 - 하지만 더 단순하게 하지 말라."**

그렇다면 이 원칙들을 실제로 어떻게 적용할 수 있는지 살펴보겠습니다.

## 관련된 상태 묶기

단일 혹은 복수의 상태 변수를 사용한는 데 있어, 확신이 서지 않을 때가 종종 있습니다.

이렇게 해야 할까요?

```ts
const [x, setX] = useState<number>(0);
const [y, setY] = useState<number>(0);
```

혹은 이렇게?

```ts
const [position, setPosition] = useState<{ x: number; y: number }>({ x: 0, y: 0 });
```

두가지 접근 모두 기술적으로 가능합니다. 그러나 만약 두 상태 변수가 항상 함께 갱신된다면, 이들을 단일 상태 변수로 단일화 하는 것이 좋은 생각입니다. 그러면 이 둘의 동기화를 잊지 않고 항상 유지할 수 있습니다. 다음 예제에서 커서를 이동하면 빨간색 점의 두 좌표가 모두 업데이트됩니다.

```tsx
import { useState } from "react";

export default function MovingDot() {
  const [position, setPosition] = useState<{ x: number; y: number }>({
    x: 0,
    y: 0,
  });

  return (
    <div
      onPointerMove={(e) => {
        setPosition({
          x: e.clientX,
          y: e.clientY,
        });
      }}
      style={{
        position: "relative",
        width: "100vw",
        height: "100vh",
      }}
    >
      <div
        style={{
          position: "absolute",
          backgroundColor: "red",
          borderRadius: "50%",
          transform: `translate(${position.x}px, ${position.y}px)`,
          left: -10,
          top: -10,
          width: 20,
          height: 20,
        }}
      />
    </div>
  );
}
```

객체 혹은 배열로 데이터를 묶는, 또 다른 경우는 얼마나 많은 상태 조각들이 필요한지 알 수 없을 때 입니다. 예를들어, 유저가 커스텀 필드를 추가할 수 있는 Form을 구현할 때 도움이 됩니다.

> ### 주의
>
> 상태 변수가 객체라면, 다른 필드를 명시적으로 복사하지 않고 [단 한개의 필드만 업데이트할 수 없다](https://react.dev/learn/updating-objects-in-state)는 것을 기억하세요. 예를 들어, 위의 예제에서 `setPosition({ x: 100 })`는 `y` 프로퍼티를 가지고 있지 않으므로 사용할 수 없습니다. 대신에 `x`만 단일로 설정하고 싶다면 `setPosition({ ...position, x: 100 })`를 사용하거나, 두개의 상태 변수로 나눈 후 `setX(100)`를 사용해야 합니다.

## 상태의 모순 피하기

다음은 `isSending`과 `isSent` 상태 변수를 사용하는 호텔의 피드백 Form입니다.

```tsx
import { useState, type FormEvent } from "react";

export default function FeedbackForm() {
  const [text, setText] = useState<string>("");
  const [isSending, setIsSending] = useState<boolean>(false);
  const [isSent, setIsSent] = useState<boolean>(false);

  async function handleSubmit(e: FormEvent<HTMLFormElement>) {
    e.preventDefault();
    setIsSending(true);
    await sendMessage(text);
    setIsSending(false);
    setIsSent(true);
  }

  if (isSent) {
    return <h1>Thanks for feedback!</h1>;
  }

  return (
    <form onSubmit={handleSubmit}>
      <p>How was your stay at The Prancing Pony?</p>
      <textarea disabled={isSending} value={text} onChange={(e) => setText(e.target.value)} />
      <br />
      <button disabled={isSending} type="submit">
        Send
      </button>
      {isSending && <p>Sending...</p>}
    </form>
  );
}

// Pretend to send a message.
function sendMessage(text) {
  return new Promise((resolve) => {
    setTimeout(resolve, 2000);
  });
}
```

이 코드는 "불가능한" 상태에 대해 가능성이 열린 상태로 실행됩니다. 예를 들어, 만약 `setIsSent`와 `setIsSending`을 함꼐 변경하는 것을 까먹는다면, `isSending`과 `isSent`가 동시에 `true`가 되는 상황에 처할 수 있습니다. 컴포넌트가 복잡할수록, 문제점을 파악하기 점점 힘들어집니다.

`isSending`과 `isSent`는 동시에 `true`가 되면 안되기 때문에, 3가지 상태중 하나를 가지는 한 개의 `status` 상태 변수로 대체하는 것이 좋습니다: `typing` (초기값), `sending`, 그리고 `sent`:

```tsx
import { useState, type FormEvent } from "react";

type Status = "typing" | "sending" | "sent";

export default function FeedbackForm() {
  const [text, setText] = useState<string>("");
  const [status, setStatus] = useState<Status>("typing");

  async function handleSubmit(e: FormEvent<HTMLFormElement>) {
    e.preventDefault();
    setStatus("sending");
    await sendMessage(text);
    setStatus("sent");
  }

  const isSending = status === "sending";
  const isSent = status === "sent";

  if (isSent) {
    return <h1>Thanks for feedback!</h1>;
  }

  return (
    <form onSubmit={handleSubmit}>
      <p>How was your stay at The Prancing Pony?</p>
      <textarea disabled={isSending} value={text} onChange={(e) => setText(e.target.value)} />
      <br />
      <button disabled={isSending} type="submit">
        Send
      </button>
      {isSending && <p>Sending...</p>}
    </form>
  );
}

// Pretend to send a message.
function sendMessage(text) {
  return new Promise((resolve) => {
    setTimeout(resolve, 2000);
  });
}
```

가독성을 위해 상수값을 선언할 수 있습니다.

```ts
const isSending = status === "sending";
const isSent = status === "sent";
```

그러나 이건 상태 변수는 아니기 떄문에, 상호 동기화가 깨지는 것에 대해 걱정하지 않아도 됩니다.

## 중복된 상태 피하기

랜더링 중 컴포넌트의 props 혹은 이미 존재하는 상태 변수를 이용해 정보를 계산할 수 있다면, 컴포넌트의 상태로 **추가하면 안됩니다.**

예시로 다음 Form을 들 수 있습니다. 동작은 하지만, 중복된 상태가 보이나요?

```tsx
import { useState, type ChangeEvent } from "react";

export default function Form() {
  const [firstName, setFirstName] = useState("");
  const [lastName, setLastName] = useState("");
  const [fullName, setFullName] = useState("");

  function handleFirstNameChange(e: ChangeEvent<HTMLInputElement>) {
    setFirstName(e.target.value);
    setFullName(e.target.value + " " + lastName);
  }

  function handleLastNameChange(e: ChangeEvent<HTMLInputElement>) {
    setLastName(e.target.value);
    setFullName(firstName + " " + e.target.value);
  }

  return (
    <>
      <h2>Let’s check you in</h2>
      <label>
        First name: <input value={firstName} onChange={handleFirstNameChange} />
      </label>
      <label>
        Last name: <input value={lastName} onChange={handleLastNameChange} />
      </label>
      <p>
        Your ticket will be issued to: <b>{fullName}</b>
      </p>
    </>
  );
}
```

이 Form은 세가지 상태 변수를 가집니다: `firstName`, `lastName`, 그리고 `fullName`. 그러나, `fullName`은 중복됩니다. 랜더 과정에서 `fullName`은 언제나 `firstName`과 `lastName`으로 계산할 수 있으므로, 상태에서 제거해야 합니다.

다음과 같이 할 수 있습니다.

```tsx
import { useState, type ChangeEvent } from "react";

export default function Form() {
  const [firstName, setFirstName] = useState("");
  const [lastName, setLastName] = useState("");

  const fullName = firstName + " " + lastName;

  function handleFirstNameChange(e: ChangeEvent<HTMLInputElement>) {
    setFirstName(e.target.value);
  }

  function handleLastNameChange(e: ChangeEvent<HTMLInputElement>) {
    setLastName(e.target.value);
  }

  return (
    <>
      <h2>Let’s check you in</h2>
      <label>
        First name: <input value={firstName} onChange={handleFirstNameChange} />
      </label>
      <label>
        Last name: <input value={lastName} onChange={handleLastNameChange} />
      </label>
      <p>
        Your ticket will be issued to: <b>{fullName}</b>
      </p>
    </>
  );
}
```

여기서, `fullName`는 상태가 아닙니다. 대신에, 랜더 과정에서 계산됩니다.

```ts
const fullName = firstName + " " + lastName;
```

결과적으로, change handler에서 이것을 갱신하기 위해 어떤 특별한 처리를 할 필요가 없어집니다. `setFirstName`나 `setLastName`를 호출하면, 리렌더가 발생하고, 최신 Form 데이터로`fullName`의 다음 값이 계산됩니다.

> ### 상태에서 prop을 미러링하지 마세요.
>
> 일반적인 복제된 상태에 대한 예시는 다음과 같습니다.
>
> ```tsx
> interface Props {
>   messageColor: string;
> }
>
> function Message({ messageColor }: Props) {
>   const [color, setColor] = useState(messageColor);
>   // ...
> }
> ```
>
> 여기서 `color` 상태 변수는 `messageColor` prop 으로 초기화됩니다. 여기서 문제는 \*\*부모 컴포넌트가 후에 다른 값의 `messageColor`를 전달한다면(예를 들어, `red` 대신에 `blue`), `color` 상태 변수는 갱신되지 않습니다. 상태는 첫 랜더 과정에서 초기화된 그대로 입니다.
>
> 이것이 상태 변수가 어떤 prop을 "미러링"하는 것이 혼란을 야기하는 이유입니다. 대신에, `messageColor` prop을 코드에서 직접 사용하세요. 만약 더 짧은 이름을 사용하고 싶다면, 상수를 사용하세요.
>
> ```tsx
> function Message({ messageColor }: Props) {
>   const color = messageColor;
> }
> ```
>
> 이 방법은 부모 컴포넌트로부터 전달 받은 prop과 동기화가 깨지지 않습니다.
>
> prop을 상태에 "미러링"하는 것은 특정 prop의 갱신을 전부 무시하고 싶을 경우에만 가능합니다. 이런 경우 일반적으로 새로운 값이 무시된다는 것을 명시하기 위해서 prop의 이름이 `initial` 혹은 `default`로 시작합니다.
>
> ```tsx
> interface Props {
>   initialColor: string;
> }
>
> function Message({ initialColor }: Props) {
>   // `color` 상태 변수는 `initialColor`의 첫번째 값에 고정됩니다.
>   // 이후의 `initialColor` prop의 변화는 무시됩니다.
>   const [color, setColor] = useState(initialColor);
> }
> ```

## 복제된 상태 피하기

메뉴 리스트 컴포넌트에서 한개의 여행 간식을 선택할 수 있습니다.

```tsx
import { useState } from "react";

interface Item {
  title: string;
  id: number;
}

const initialItems: Array<Item> = [
  { title: "pretzels", id: 0 },
  { title: "crispy seaweed", id: 1 },
  { title: "granola bar", id: 2 },
];

export default function Menu() {
  const [items, setItems] = useState<Array<Item>>(initialItems);
  const [selectedItem, setSelectedItem] = useState<Item>(items[0]);

  return (
    <>
      <h2>What's your travel snack?</h2>
      <ul>
        {items.map((item) => (
          <li key={item.id}>
            {item.title}{" "}
            <button
              onClick={() => {
                setSelectedItem(item);
              }}
            >
              Choose
            </button>
          </li>
        ))}
      </ul>
      <p>You picked {selectedItem.title}.</p>
    </>
  );
}
```

현재, 선택된 아이템을 객체 상태로 `selectedItem` 상태 변수에 저장합니다. 그러나, **`selectedItem`의 내용물이 `items` 리스트 내부 아이템중 하나와 같은 객체인 것**은 좋지 않습니다. 이건, 아이템에 관한 정보가 두 곳에 복제됐다는 뜻입니다.

왜 이것이 문제가 될까요? 아이템을 수정 가능하게 맍들어 봅시다.

```tsx
import { useState, type ChangeEvent } from "react";

interface Item {
  title: string;
  id: number;
}

const initialItems: Array<Item> = [
  { title: "pretzels", id: 0 },
  { title: "crispy seaweed", id: 1 },
  { title: "granola bar", id: 2 },
];

export default function Menu() {
  const [items, setItems] = useState<Array<Item>>(initialItems);
  const [selectedItem, setSelectedItem] = useState<Item>(items[0]);

  function handleItemChange(id: Item["id"], e: ChangeEvent<HTMLInputElement>) {
    setItems(
      items.map((item) => {
        if (item.id === id) {
          return {
            ...item,
            title: e.target.value,
          };
        } else {
          return item;
        }
      })
    );
  }

  return (
    <>
      <h2>What's your travel snack?</h2>
      <ul>
        {items.map((item, index) => (
          <li key={item.id}>
            <input
              value={item.title}
              onChange={(e) => {
                handleItemChange(item.id, e);
              }}
            />{" "}
            <button
              onClick={() => {
                setSelectedItem(item);
              }}
            >
              Choose
            </button>
          </li>
        ))}
      </ul>
      <p>You picked {selectedItem.title}.</p>
    </>
  );
}
```

아이템의 "Choose"를 먼저 클릭하고 이것을 수정하면, input은 갱신되지만 하단의 레이블은 수정을 반영하지 못하는 것에 주목하세요. 이건 상태를 복제하고, `selectedItem`의 갱신을 깜빡했기 때문입니다.

비록 `selectedItem`를 같이 갱신할 수 있더라도, 복제를 제거하는 것이 더 쉬운 수정 방법입니다. 이 예시에서, `selectedItem` 객체 대신에(`items` 내부의 객체로부터 복제되어 생성된), `selectedId`를 상태로 사용하고, `items` 배열에서 해당 ID로 `selectedItem`를 찾아서 가져옵니다.

```tsx
import { useState, type ChangeEvent } from "react";

interface Item {
  title: string;
  id: number;
}

const initialItems: Array<Item> = [
  { title: "pretzels", id: 0 },
  { title: "crispy seaweed", id: 1 },
  { title: "granola bar", id: 2 },
];

export default function Menu() {
  const [items, setItems] = useState<Array<Item>>(initialItems);
  const [selectedId, setSelectedId] = useState<Item["id"]>(0);

  const selectedItem = items.find((item) => item.id === selectedId);

  function handleItemChange(id: Item["id"], e: ChangeEvent<HTMLInputElement>) {
    setItems(
      items.map((item) => {
        if (item.id === id) {
          return {
            ...item,
            title: e.target.value,
          };
        } else {
          return item;
        }
      })
    );
  }

  return (
    <>
      <h2>What's your travel snack?</h2>
      <ul>
        {items.map((item, index) => (
          <li key={item.id}>
            <input
              value={item.title}
              onChange={(e) => {
                handleItemChange(item.id, e);
              }}
            />{" "}
            <button
              onClick={() => {
                setSelectedId(item.id);
              }}
            >
              Choose
            </button>
          </li>
        ))}
      </ul>
      <p>You picked {selectedItem.title}.</p>
    </>
  );
}
```

(또는, 선택된 index를 상태로 가지고 있을 수 있습니다.)

상태는 다음과 같이 복제되어 사용되었었습니다.

- `items = [{ id: 0, title: 'pretzels'}, ...]`
- `selectedItem = {id: 0, title: 'pretzels'}`

그러나, 다음과 같이 변경됐습니다.

- `items = [{ id: 0, title: 'pretzels'}, ...]`
- `selectedId = 0`

복제는 업어졌고, 필수적인 상태만 유지하게 됐습니다.

이제 선택된 아이템을 수정하더라도, 하단의 메시지도 즉시 갱신됩니다. `setItems`이 리랜더를 일으키고, `items.find(...)`이 갱신된 제목을 가진 아이템을 찾기 때문입니다. 선택된 아이템 ID가 필요했기 때문에, 선택된 아이템을 상태로 가질 필요가 없었습니다.

## 깊이 중첩된 상태 피하기

행성, 대륙, 그리고 나라들로 구성된 여행 계획을 상상해보세요. 중첩된 객체와 배열을 이용해 상태로 구조화 하고 싶을겁니다. 다음 예제 처럼요.

```ts
// places.ts

export interface Place {
  id: number;
  title: string;
  childPlaces: Array<TrablePlan>;
}

export const initialTravelPlan: Place = {
  id: 0,
  title: "(Root)",
  childPlaces: [
    {
      id: 1,
      title: "Earth",
      childPlaces: [
        {
          id: 2,
          title: "Africa",
          childPlaces: [
            {
              id: 3,
              title: "Botswana",
              childPlaces: [],
            },
            // ...
          ],
        },
        // ...
      ],
    },
    // ...
  ],
};
```

```tsx
// App.tsx

import { useState } from "react";
import { initialTravelPlan, type Place } from "./places";

interface PlaceTreeProps {
  place: Place;
}

function PlaceTree({ place }: PlaceTreeProps) {
  const childPlaces = place.childPlaces;
  return (
    <li>
      {place.title}
      {childPlaces.length > 0 && (
        <ol>
          {childPlaces.map((place) => (
            <PlaceTree key={place.id} place={place} />
          ))}
        </ol>
      )}
    </li>
  );
}

export default function TravelPlan() {
  const [plan, setPlan] = useState<Place>(initialTravelPlan);
  const planets = plan.childPlaces;
  return (
    <>
      <h2>Places to visit</h2>
      <ol>
        {planets.map((place) => (
          <PlaceTree key={place.id} place={place} />
        ))}
      </ol>
    </>
  );
}
```

이제 이미 방문했던 장소를 제거할 수 있는 버튼을 추가해봅시다. 어떻게 하시겠습니까? [중첩된 상태를 갱신하기](https://react.dev/learn/updating-objects-in-state#updating-a-nested-object) 위해서는 객체의 변경된 부분으로부터 위쪽의 모든 것들을 복제해야 합니다. 깊이 중첩된 곳을 삭제하는 것은 부모의 place 체인 전체를 복제해야 합니다. 이런 코드는 굉장히 장황해질 수 있습니다.

**갱신을 쉽게 하기에는 상태가 너무 중첩돼있다면, "평평하게" 만드는 것을 고려해보세요.** 여기 데이터를 재구축하는 한가지 방법이 있습니다. 모든 `place`가 자식 place를 배열로 갖는 트리 같은 구조 대신에, 각 place가 자식 place ID를 배열로 갖게 할 수 있습니다. 그리고 각 place ID와 관련된 place를 맵핑하여 저장합니다.

```ts
// places.ts

export type PlaceID = number;

export interface Place {
  id: PlaceID;
  title: string;
  childPlaces: Array<TrablePlan>;
}

export type PlacesById = Record<PlaceID, Place>;

export const initialTravelPlan: PlacesById = {
  0: {
    id: 0,
    title: "(Root)",
    childIds: [1, 43, 47],
  },
  1: {
    id: 1,
    title: "Earth",
    childIds: [2, 10, 19, 27, 35],
  },
  2: {
    id: 2,
    title: "Africa",
    childIds: [3, 4, 5, 6, 7, 8, 9],
  },
  // ...
};
```

```tsx
// App.tsx

import { useState } from "react";
import { initialTravelPlan, type PlaceID, type Place, type PlacesById } from "./places";

interface PlaceTreeProps {
  id: PlaceID;
  placesById: PlacesById;
}

function PlaceTree({ id, placesById }: PlaceTreeProps) {
  const place = placesById[id];
  const childIds = place.childIds;
  return (
    <li>
      {place.title}
      {childPlaces.length > 0 && (
        <ol>
          {childPlaces.map((place) => (
            <PlaceTree key={place.id} place={place} />
          ))}
        </ol>
      )}
    </li>
  );
}

export default function TravelPlan() {
  const [plan, setPlan] = useState<PlacesById>(initialTravelPlan);
  const root = plan[0];
  const planetIds = root.childIds;
  return (
    <>
      <h2>Places to visit</h2>
      <ol>
        {planetIds.map((id) => (
          <PlaceTree key={id} id={id} placesById={plan} />
        ))}
      </ol>
    </>
  );
}
```

이제 상태가 "평평해"("정규화"라고도 합니다)졌고, 중첩된 아이템들을 갱신하기 쉬워졌습니다.

이제 place를 제거하기 위해서, 두 단계의 상태만 갱신하면 됩니다.

- 갱신된 버전의 부모 place의 `childIds`에서 제거된 ID를 제외해야합니다.
- 갱신된 버전의 "table" 객체는 갱신된 버전의 부모 place를 가지고 있어야 합니다.

여기 어떻게 할 수 있는지에 대한 예시가 있습니다.

```tsx
// App.tsx

import { useState } from "react";
import { initialTravelPlan, type PlaceID, type Place, type PlacesById } from "./places";

interface PlaceTreeProps {
  id: PlaceID;
  parentId: PlaceID;
  placesById: PlacesById;
  onComplete: (parentId: PlaceID, id: PlaceID) => void;
}

function PlaceTree({ id, parentId, placesById, onComplete }: PlaceTreeProps) {
  const place = placesById[id];
  const childIds = place.childIds;
  return (
    <li>
      {place.title}
      <button
        onClick={() => {
          onComplete(parentId, id);
        }}
      >
        Complete
      </button>
      {childIds.length > 0 && (
        <ol>
          {childIds.map((childId) => (
            <PlaceTree key={childId} id={childId} parentId={id} placesById={placesById} onComplete={onComplete} />
          ))}
        </ol>
      )}
    </li>
  );
}

export default function TravelPlan() {
  const [plan, setPlan] = useState<PlacesById>(initialTravelPlan);

  function handleComplete(parentId: PlaceID, childId: PlaceID) {
    const parent = plan[parentId];
    // 새로운 버전의 부모 place를 생성합니다.
    // 삭제된 자식의 ID를 포함하지 않습니다.
    const nextParent = {
      ...parent,
      childIds: parent.childIds.filter((id) => id !== childId),
    };
    // 상태 객체를 갱신합니다.
    setPlan({
      ...plan,
      // ...업데이트된 부모를 가질 수 있도록 합니다.
      [parentId]: nextParent,
    });
  }

  const root = plan[0];
  const planetIds = root.childIds;
  return (
    <>
      <h2>Places to visit</h2>
      <ol>
        {planetIds.map((id) => (
          <PlaceTree key={id} id={id} parentId={0} placesById={plan} onComplete={handleComplete} />
        ))}
      </ol>
    </>
  );
}
```

원하는 만큼 상태를 중첩할 수 있지만, "평평하게" 만들면 수 많은 문제를 해결할 수 있습니다. 상태를 갱신하기 쉽게 만들며, 중첩된 객체의 다른 부분에서 중첩이 발샐하지 않도록 해줍니다.

때때로, 중첩된 상태를 자식 컴포넌트 안으로 이동시켜 상태 중첩을 줄일 수도 있습니다. 이 방식은 아이템이 호버 됐는지와 같은, 저장할 필요가 없는 일시적인 UI 상태에 유용합니다.

## 요약

- 두 가지 상태 변수가 항상 함께 갱신된다면, 하나로 합치는 것을 고려해보세요.
- "불가능한" 상태 생성을 피하기 위해 상태 변수를 신중하게 선택하세요.
- 업데이트 할 때 실수를 줄일 수 있는 방법으로 상태를 구성하세요.
- 장황하고 중복된 상태를 피하면 동기화를 유지하지 않아도 됩니다.
- 특별히 갱신을 막고자하는 것이 아니라면 prop을 상태에 넣지 마세요.
- 선택하기와 같은 UI 패턴에서, 객체 자체보다는 ID나 index를 상태로 사용하세요.
- 만약 깊게 중첩된 상태를 갱신하기 복잡하다면, 이를 평평하게 해보세요.
