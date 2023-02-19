# 렌더와 커밋

> 원문: [React Docs Beta > Learn > Learn React > Render and Commit](https://beta.reactjs.org/learn/keeping-components-pure)

컴포넌트가 화면에 출력되기 전에 반드시 React로부터 렌더됩니다. 이 과정의 단계들을 이해한다면 자신의 코드가 어떻게 실행되는지 상상하고 동작을 설명하는데 도움이 될겁니다.

> 이런 것을 배워요
>
> - React에서 렌더의 의미
> - 언제 그리고 왜 React가 컴포넌트를 렌더하는지
> - 컴포넌트를 화면에 출력하는 과정
> - 렌더링이 항상 DOM을 업데이트하지 않는 이유

컴포넌트가 부엌에서 재료들로 맛있는 요리를 조합하는 요리사라고 상상하세요. 이 시나리오에서, React는 고객의 요청을 받고 전달하는 웨이터입니다. 요청을 받고 UI를 서빙하는 과정은 3단계로 이루어집니다.

- 렌더 유발하기(Triggering) (손님의 주문은 부엌으로 전달합니다)
- 컴포넌트 렌더하기 (부엌에서 음식을 준비합니다)
- DOM에 커밋하기 (주문한 음식을 테이블어 놓습니다)

## Step1: 렌더를 유발합니다.

컴포넌틀를 렌더하는 이유는 두가지가 있습니다.

- 컴포넌트 초기화 렌더
- 컴포넌트(혹은 해당 컴포넌트의 조상)의 상태(state)가 업데이트 됐을 떄

### 초기화 렌더

앱이 시작될 때, 초기화 렌더를 발생시켜야 합니다. 때때로 프렘이워크나 센드박스는 이 코드를 숨기지만, DOM 노드를 타겟으로 [`createRoot`](https://beta.reactjs.org/apis/react-dom/client/createRoot) 호출을 끝낸 다음에 컴포넌트의 `render` 메서드를 호출합니다.

```tsx
import Image from "./Image.js";
import { createRoot } from "react-dom/client";

const root = createRoot(document.getElementById("root"));
root.render(<Image />);
```

`root.render()` 호출을 주석처리하면 컴포넌트가 사라집니다.

### 상태(State) 업데이트에 따른 리렌더

컴포넌트의 초기화 렌더가 끝나면, [set 함수](https://beta.reactjs.org/reference/react/useState#setstate)로 컴포넌트의 상태(State)를 업데이트하여 추가적은 렌더를 일으킬 수 있습니다. 컴포넌트 업데이트는 자동으로 렌더를 대기열에 추가합니다. (레스토랑에서 손님이 첫 주문 후에 배고픔 혹은 목마름 상태에 따라 차, 디저트 등을 주문하는 것으로 상상할 수 있습니다.)

## Step2: React가 컴포넌트를 렌더합니다.

렌더가 유발된 뒤, React는 화면에 무엇을 그려야하는지 알아내기 위해 컴포넌트들을 호출합니다. **"Rendering"은 React가 컴포넌트르 호출하는 것 입니다.**

- 초기화 렌더에서, React는 루트 컴포넌트를 호출합니다.
- 후속 렌더에서, 리엑트는 업데이트돼서 렌더를 야기한 상태를 가진 컴포넌트 함수를 호출합니다.

이 과정은 재귀적입니다. 만약 업데이트된 컴퐁넌트가 다른 컴포넌트를 반환하면, React는 그 컴포넌트를 다음에 렌더하고, 그리고 만약 그 컴포넌트가 또 다른 무언가를 반환하면, 그 컴포넌트는 다음에 렌더되는 식으로 이어집니다. 이 과정은 중첩된 컴포넌트가 더 이상 없고 React가 화면에 정확하게 무엇을 그려야하는지 알기 전까지 계속됩니다.

다음 예제에서 React는 `Gallery()`를 호출하고 `Image()`를 다수 호출합니다.

```tsx
export default function Gallery() {
  return (
    <section>
      <h1>Inspiring Sculptures</h1>
      <Image />
      <Image />
      <Image />
    </section>
  );
}

function Image() {
  return (
    <img
      src="https://i.imgur.com/ZF6s192.jpg"
      alt="'Floralis Genérica' by Eduardo Catalano: a gigantic metallic flower sculpture with reflective petals"
    />
  );
}
```

- 초기화 렌더 동안에, React는 `<section>`, `<h1>`, 그리고 세 개의 `<img>` 태그의 [DOM 노드를 생성합니다.](https://developer.mozilla.org/docs/Web/API/Document/createElement)
- 리랜더 과정에서, React는 이전 렌더 이후로 변경된 프로퍼티가 있다면 이들의 계산합니다.

> ### 주의 사항
>
> 렌더는 항상 순수한 계산이여야 합니다.
>
> - **동일 입력, 동일 출력.** 같은 입력이 주어지면, 컴포넌트는 항상 같은 JSX를 반환해야 합니다. (만약 누군가 토마토가 들어간 셀러드를 주문했을 때, 양파가 들어간 셀러드를 받으면 안됩니다!)
> - **스스로의 비즈니스에만 관여합니다.** 렌더링 이전에 존재하던 변수나 객체를 변경해서는 안됩니다. (주문은 다른 누군가의 주문을 변경해서는 안됩니다.)
>
> 그렇지 않으면, 코드 베이스가 점점 복잡하게 성장하면서 혼란스러운 버그와 예기치 못한 행동을 직면할 수 있습니다. " Strict Mode"로 개발한다면, React는 각 컴포넌트 함수르 두 번씩 호출해서, 더러운 함수로부터 야기되는 표면적인 드러내는 것을 도와줍니다.

> ### 성능 최적화
>
> 업데이트 된 컴포넌트가 트리의 상위에 위차한 경우 중첩된 컴포넌트를 모두 렌더링하는 기본 동작은 성능에 있어 최선이 아닙니다. 만약 성능적인 문제를 겪게 된다면, [성능 섹션](https://reactjs.org/docs/optimizing-performance.html)에서 말하는 몇가지 옵션이 있습니다. **너무 이르게 최적화 하지는 마세요!**

## Step3: React가 변경사항을 DOM에 커밋합니다.

컴포넌트를 렌더링(호출)한 후에, React는 DOM을 수정할 겁니다.

- **초기화 렌더에서**, React는 화면에 생성된 모든 DOM 노드들을 넣기 위해 [`appendChild()`](https://developer.mozilla.org/docs/Web/API/Node/appendChild) DOM API를 사용합니다.
- **리렌더에서**, React는 DOM을 최신 렌더 결과와 일치시기키 위해 최소한으로 필요한 명령들(렌더링 동안에 계산된)을 적용합니다.

**React는 DOM 노드들을 렌더와 차이가 있느 경우에만 변경합니다.** 예를 들어, 매 초마다 부모로부터 다른 props를 전달받아 리렌더되는 컴포넌트가 있습니다. 어떻게 컴포넌트가 리렌더 될 때 텍스트가 사라지지 않으면서, `<input`에 텍스트를 추가하고 `value`를 업데이트 할 수 있는지 생각해보세요.

```tsx
export default function Clock({ time }) {
  return (
    <>
      <h1>{time}</h1>
      <input />
    </>
  );
}
```

이 마지막 단계에서 React는 오직 `<h1>`의 내용만을 새로운 `time`으로 업데이트하기 떄문에 가능합니다. JSX에서 `<input>`은 이전 시간과 동일한 위치에 나타나기 때문에, React는 `<input>` 혹은 `input`의 값을 변경하지 않습니다!

## Epilogue: 브라우저 페인트

렌더링이 끝나고 React가 DOM을 업데이트 한 후에, 브라우저는 화면을 다시 그립니다. 비록 이 과정을 "browser rendering"이라 알고 있을지라도, 우리는 이 문서의 남은 부분과의 혼란을 피하기 위해 "painting"라고 부릅니다.

## 요약

- React 앱에서 화면 업데이트는 3 단계로 이루어진다.
  1. Trigger
  2. Render
  3. Commit
- Strict Mode를 사용해서 컴포넌트에서의 실수를 찾을 수 있습니다.
- React는 렌더링 결과가 이전과 동일하다면 DOM을 건드리지 않습니다.
