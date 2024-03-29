# 컴포넌트 순수하게 유지하기

> 원문: [React Docs Beta > Learn > Learn React > Keeping Components Pure](https://beta.reactjs.org/learn/keeping-components-pure)

어떤 Javascript 함수는 순수합니다. 순수 함수는 계산 외에 다른것을 수행하지 않습니다. 컴포넌트를 엄격하게 순수 함수로만 작성하려면, 코드 베이스가 커지면서 나타나는 당혹스러운 버그나 예츠하지 못한 동작을 피할 수 있습니다. 이런 이점을 취하기 위해, 반드시 따라야하는 몇가지 구칙이 있습니다.

> 이런 것을 배워요
>
> 순수가 무엇이며 어떻게 버그를 회피하는데 도움이 되는지
> 렌더 단계의 변경을 제외하여 컴포넌트를 순수하게 유지하는 방법
> Strict Mode를 사용해서 컴포넌트의 실수를 찾는 방법

## 순수: 공식같은 컴포넌트

컴퓨터 공학에서(특히 함수형 프로그래밍 세계에서) [순수 함수](https://en.wikipedia.org/wiki/Pure_function)는 다음 특징들을 지니는 함수입니다.

- **자신의 비즈니스만을 신경씁니다.** 호출되기 전부터 존재하던 객체나 변수를 변경하지 않습니다.
- **동일 입력, 동일 출력.** 동일한 입력에대해, 순수 함수는 언제나 동일한 결과를 출력합니다.

아마 순수 함수의 예제중 하나와 이미 친숙할 겁니다: 수학의 공식입니다.

다음 수학 공식을 생각해봅시다: y = 2x

만약 x = 2이면, 항상 y = 4입니다.

만약 x = 3이면, 항상 y = 6입니다.

만약 x = 3일 때, 시간이나 주식 시장의 상태에 따라 y가 -1 혹은 2.5가 되는 경우는 없습니다.

y = 2x일 때 x = 3이면, y는 언제나 6이 될 것입니다.

Javascript 함수로 표현하면 다음과 같을겁니다.

```typescript
function double(number: number): number {
  return 2 * number;
}
```

위 예제에서, `double`은 순수 함수입니다. `3`을 넘기면, 언제나 `6`이 반환됩니다.

리엑트는 이 컨셉을 기반으로 설계됐습니다. **React는 작성된 모든 컴포넌트가 순수 함수일 것이라고 가정합니다.** 이 말은 즉, React 컴포넌트는 동일한 입력에 대해 항상 동일한 JSX를 반환해야 한다는 것 입니다.

```tsx
interface RecipeProps {
  drinkers: number;
}

function Recipe({ drinkers }: RecipeProps) {
  return (
    <ol>
      <li>Boil {drinkers} cups of water.</li>
      <li>
        Add {drinkers} spoons of tea and {0.5 * drinkers} spoons of spice.
      </li>
      <li>Add {0.5 * drinkers} cups of milk to boil and sugar to taste.</li>
    </ol>
  );
}

export default function App() {
  return (
    <section>
      <h1>Spiced Chai Recipe</h1>
      <h2>For two</h2>
      <Recipe drinkers={2} />
      <h2>For a gathering</h2>
      <Recipe drinkers={4} />
    </section>
  );
}
```

`Recipe`에 `drinkers={2}`을 넘기면, 항상 `2 cups of water`을 포함한 JSX를 반환할 것 입니다.

만약 `drinkers={4}`를 넘긴다면, 항상 `4 cups of water`을 포함한 JSX를 반환할 겁니다.

마치 수학의 공식과 같이 말입니다.

컴포넌트를 레시피와 같다고 생가할 수 있습니다. 만약 레시피를 따라하며 그 과정에서 새로운 재료를 사용하지 않는다면, 매번 같은 음식을 만들겁니다. 여기서 음식은 렌더하기 위해 컴포너트가 React에 제공하는 JSX와 같슴니다.

## 사이드 이펙트: 의도(하지 않은) 결과

React의 렌더링 과정은 언제나 순수해야 합니다. 컴포넌트는 항상 자신만의 JSX를 반환하고, 렌더링 전에 존재하던 어떤 객체나 변수도 변경하지 않습니다. 이런 변경은 컴포넌트를 순수하지 않게 만듭니다.

다음은 이 구칙을 어긴 컴포넌트입니다.

```tsx
let guest: number = 0;

function Cup() {
  // 나쁨: 이미 존재하던 변수를 변경합니다.
  guest = guest + 1;
  return <h2>Tea cup for guest #{guest}</h2>;
}

export default function TeaSet() {
  return (
    <>
      <Cup />
      <Cup />
      <Cup />
    </>
  );
}
```

이 컴포넌트는 밖에서 선언된 `guest` 변수를 읽고 수정합니다. 이것은 **이 컴포넌트를 여러번 호출할 때마다 서로 다른 JSX를 생성**한다는 의미입니다! 게다가, 다른 컴포넌트에서 `guest`를 읽는다면, 렌더 될 때마다 이들조차 다른 JSX를 생성할 겁니다. 이는 예측 가능하지 않습니다.

우리의 y = 2x로 돌아간다면, 이제 x = 2일 떄 y = 4라는 것을 믿지 못하게 됐습니다. 테스트는 실패할 수 있고, 유저는 당황하게 될 것이며, 비행기는 하늘에서 추락할 것입니다. 이것이 어떻게 혼란스러운 버그러 이어지는지 확일할 수 있습니다.

`guest`를 props으로 넘기는 것으로 컴포넌트를 수정할 수 있습니다.

```tsx
interface CupProps {
  guest: number;
}

function Cup({ guest }: CupProps) {
  return <h2>Tea cup for guest #{guest}</h2>;
}

export default function TeaSet() {
  return (
    <>
      <Cup guest={1} />
      <Cup guest={2} />
      <Cup guest={3} />
    </>
  );
}
```

이제 컴포넌트는 순수하며, 반환되는 JSX는 오직 `guest` prop에만 의존합니다.

일반적으로, 컴포넌트들이 특정한 순서대로 렌더될 것이라고 예상해서는 안됩니다. y = 2x를 y = 5x 이전에 호출하던 이후에 호출하던 상관 없습니다. 두 공식은 각자 독립적으로 수행됩니다. 이와 같이, 각 컴포넌트는 "자신만을 생각"해야하며, 렌더링 동안에 다른 컴포넌트에 의존하거나 간섭하려 시도하지 않슴니다. 렌더링은 학교 시험과 같습니다. 각 컴포넌트는 자신의 JSX를 계산합니다.

> ### StrictMode로 순수하지 않은 계산 감지하기
>
> 아직 다 사용하진 않았겠지만, React에는 렌더링 도중에 읽을수 있는 입력값이 세 종류 있습니다. `props`, `state` 그리고 `context`입니다. 이 세가지 입력은 언제나 읽기 전용으로 취급해야 합니다.
>
> 사용자 입력에 대한 응답으로 무언갈 변경하고 싶을 때, 변수에 입력하기 보다는 set state를 사용해야 합니다. 컴포넌트가 렌더링 되기 이전에 이미 존재하던 객체나 변수를 변경해서는 안됩니다.
>
> React는 개발중 컴포넌트의 함수를 두번씩 호출하는 "Strict Mode"를 제공합니다. 컴포넌트 함수를 두번 호출하므로 인해, Strict Mode는 규칙을 위반하는 컴포넌트를 찾는 것을 도와줍니다.
>
> 원문 예제에서 어떻게 “Guest #2”, “Guest #4”, 그리고 “Guest #6” 대신에 “Guest #2”, “Guest #4”, 그리고 “Guest #6”가 출력됐는지 생각해 보세요. 원문의 함수는 순수하지 않고, 두 번씩 호출한다면 정상 작동하지 않습니다. 하지만 순수하게 고쳐진 버전에서는 매번 두 번씩 호출한다고 해도 정상적으로 작동합니다. 순수 함수는 단순히 계산만 하기 때문에, 두 번씩 호출하는 것이 어떤 변화도 일으키지 않습니다. `double(2)`을 두 번 호출한다고 해서 출력이 변하지 않고, y = 2x를 두 번 계산한다 해서 y가 무엇인지 변하지 않는 것과 같습니다. 항상 동일 입력에는 동일한 결과가 있습니다.
>
> Strict Mode는 production에 영향을 주지 않기 떄문에, 앱을 느리게 만들지 않습니다. Strict Mode로 전환하려면, 루트 컴포넌트를 `<React.StrictMode>`로 감싸면 됩니다. 몇몇 Framework는 이를 디폴트로 가져갑니다.

### Local Mutaion: 컴포넌트의 작은 비밀

위의 예제에서, 컴포넌트가 렌더링 과정에서 사전에 존재하던 변수를 변경한 것이 문제였습니다. 조금 더 무섭게 들리게 하기 위해 이것을 종종 "mutaion"이라고 불립니다. 순수 함수는 함수 스코프 밖의 변수나 호출 전에 존재하던 객체를 변화시키지(mutate) 않습니다. 이것은 함수를 순수하지 않게 하기 떄문입니다.

그러나, **렌더링 과정에서 생성된 번수나 객체는 변경해도 괜찮습니다.** 다음 예제에서, `[]` 배열을 생성하고, `cups` 변수에 할당하며, 12개의 컵들을 `push`합니다.

```tsx
interface CupProps {
  guest: number;
}

function Cup({ guest }: CupProps) {
  return <h2>Tea cup for guest #{guest}</h2>;
}

export default function TeaGathering() {
  let cups = [];
  for (let i = 1; i <= 12; i++) {
    cups.push(<Cup key={i} guest={i} />);
  }
  return cups;
}
```

만약 `cups` 변수 혹은 `[]` 배열이 `TeaGathering` 함수 밖에서 생성됐다면, 이건 아주 큰 문제가 됐을겁니다! 배열에 아이템을 푸쉬하면서 사전에 존재하던 객체를 변경하게 됩니다.

그러나, `TeaGathering` 내부에서 동일한 렌더 과정 중 생성했기 때문에 괘찮습니다. `TeaGathering` 외부의 어떤 코드도 이 과정을 알지 못합니다. 이것을 **“local mutation”**이라 부르며, 컴포넌트가 가지는 작은 비밀과 같습니다.

## 사이드 이펙트를 어디서 일으킬 수 있는가

함수형 프로그래밍은 순수성에 크게 의존하지만, 언젠가 어디에서는 무언가가 변경돼야 합니다. 이것이 프로그래밍의 본질입니다. 화면을 업데이트하고, 에니메이션을 시작하고, 데이터를 변경하는 것과 같은 변경들은 사이드 이펙트(Side Effect)라고 불립니다. 이것들은 렌더링 과정에서 일어나는 것이 아니라, "측면(side)에서" 발생하기 때문입니다.

React에서, 사이드 이펙트는 일반적으로 이벤트 핸들러(Event Handler)에서 시작합니다. 이벤트 핸들러는 React가 어떤 액션을 수행할때 실행하는 함수입니다. 예를 들어, 버튼 클릭하는 순간처럼 말입니다. 비록 이벤트 핸들러가 컴포넌트 내부에 정의됐다 할지라도, 렌더링 과정에서 실행되지 않습니다. **그렇기 떄문에 이벤트 핸들러는 순수할 필요가 없습니다.**

만약 사이드 이펙트를 위한 적절한 이벤트 핸들러를 찾지 못하고 지쳐버렸다면, 컴포넌트 내부에 `useEffect`를 호출해서 반환된 JSX에 결합시킬 수 있습니다. `useEffect`는 React에게 렌더링 후 사이드 이펙트가 허락되는 순간에 실핼시켜달라 말해줍니다. **하지만, 이 방식은 최후의 보루여야 합니다.**

가능하다면, 렌더링 만으로 로직을 표현하려고 시도해보세요. 이것이 당신을 얼마나 멀리 데려다줄지 놀라게 될겁니다!

> ### React는 왜 순수에 신경쓰는가?
>
> 순수 함수를 쓰는 것은 약간의 습관과 훈련을 필요로 합니다. 하지만 놀라운 기회를 제공해주기도 합니다.
>
> - 컴포넌트가 다양한 환경에서 실행될 수 있습니다. 예를 들어, 서버에서 말입니다! 동일한 입력에 동일한 결과를 출력하기만 한다면, 하나의 컴포넌트는 많은 유저의 요청에 사용될 수 있습니다.
> - 입력값이 변하지 않는 [랜더링 생략 컴포넌트](https://beta.reactjs.org/reference/react/memo)(Skipping Rendering Component)를 사용하여 성능을 향상시킬 수 있습니다. 순수 함수는 항상 동일한 결과를 반환하기 때문에 캐시되어도 안전하기 때문입니다.
> - 만약 렌더링 중간에 컴포넌트 트리 깊은 곳에서 어떤 데이터가 변경되면, React는 기존의 렌더링을 끝내기 위해 시간을 낭비하지 않고 렌더링을 다시 시작할 수 있습니다. 순수성은 언제든지 계산이 중단돼도 안전하도록 만듭니다.
>
> 구현되는 React의 모든 새로운 기능은 순수성의 이점을 활용합니다. 데이터 패칭부터 에니메이션, 성능에 이르기까지, 컴포넌트를 순수하게 유지하는 것은 React 패러다임의 힘을 발현할 수 있게 해줍니다.

## 요약

- 컴포넌트는 순수해야 합니다.
  - **자신의 비즈니스만을 신경씁니다.** 호출되기 전부터 존재하던 객체나 변수를 변경하지 않습니다.
  - **동일 입력, 동일 출력.** 동일한 입력에대해, 순수 함수는 언제나 동일한 결과를 출력합니다.
- 렌더링은 언제든 일어날 수 있기 때문에, 컴포넌트들은 서로의 렌더링 순서에 의존하면 안됩니다.
- 컴포넌트가 렌더링에 사용하는 어떠한 입력도 변화시키면 안됩니다. 여기에는 props, state, 그리고 context가 포함됩니다. 화면은 업데이트하기 위해서, 사전에 존재하는 object를 변경하는 대신에 set state를 사용해야 합니다.
- 컴포넌트의 로직을 반환하는 JSX 내부에 표현하려 노력해야 합니다. "무언가를 변경"해야 할 때, 일반적으로 이벤트 핸들러 수행해야 합니다. 최후의 수단으로, `useEffect`를 사용할 수 있습니다.
- 순수 함수를 작성하는 것은 연습이 조금 필요하지만, React 패러다임의 힘을 발현할 수 있게 해줍니다.
