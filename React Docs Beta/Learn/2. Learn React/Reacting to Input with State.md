# 상태(State)로 입력에 반응하기

> 원문: [React Docs Beta > Learn > Learn React > Reacting to Input with State](https://react.dev/learn/reacting-to-input-with-state)

React는 선언적인 방법을 사용해 UI를 관리합니다. UI 각 부분을 직접 조작하는 대신에, 컴퍼넌트가 가질 수 있는 다양한 상태(State)를 기억하고, 유저의 입력에 따라 이를 전환하기만 하면 됩니다. 디자이너가 UI를 생각하는 방식과 유사합니다.

> ### 이런 것을 배워요
>
> - 선언형 UI 프로그래밍과 명령형 UI 프로그래밍의 차이점
> - 컴포넌트가 가질 수 있는 다양한 시각적 상태를 열거하는 방법
> - 코드에서 서로 다른 시각적 상태간의 변경을 트리거하는 방법

## 선언적 UI와 명령형 UI를 비교하는 방법

UI 인터렉션을 디자인할 때, 대개 유저의 행동에 대한 반응으로 UI가 어떻게 바뀌어야 하는지 생각할 겁니다. 유저가 답을 제출하는 Form을 생각해 봅시다.

- Form에 무엇인가 타이핑할 떄, "Submit" 버튼이 **활성화 된다.**
- "Submit" 버튼을 누르면, Form과 버튼이 **비활성화되고,** 스피너가 **등장한다.**
- 네트워크 요청이 성공하면, From이 **숨겨지고**, “Thank you” 메시지가 **등장한다.**
- 만약 네트워크 요청이 실패한다면, 에러 메시지가 **등장하고**, Form이 다시 **활성화된다.**

명령형 프로그래밍에서는, 인터렉션을 구현하는 방법과 위의 내용과 직접적으로 일치합니다. 방금 발생한 일에 따라 UI를 조작하기 위해서는 정확한 인터렉션 작성해야 합니다. 예를 들자면: 자동차 안에서 운전자 옆에 앉아 어디로 가야하는지 차레대로 설명해 준다고 생각해 보세요.

운전자는 당신이 어디로 가고 싶은지 모르며, 그저 당시의 지시에 따릅니다. (그리고 만약 잘못된 길을 알려준다면, 틀린 장소에 도착하게 될겁니다!). 이런 방식은 어떻게 UI를 업데이트해야 하는지, 스피너에서 버튼에 이르기까지, 각 요소를 컴퓨터에게 "명령" 해야 하기 때문에 명령형이라 불립니다.

아래 명령형 UI 프로그래밍의 예시에서, Form은 리엑트를 사용하지 않고 구현됐습니다. 오직 브라우저 [DOM](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model)을 사용했습니다.

```js
async function handleFormSubmit(e) {
  e.preventDefault();
  disable(textarea);
  disable(button);
  show(loadingMessage);
  hide(errorMessage);
  try {
    await submitForm(textarea.value);
    show(successMessage);
    hide(form);
  } catch (err) {
    show(errorMessage);
    errorMessage.textContent = err.message;
  } finally {
    hide(loadingMessage);
    enable(textarea);
    enable(button);
  }
}

function handleTextareaChange() {
  if (textarea.value.length === 0) {
    disable(button);
  } else {
    enable(button);
  }
}

function hide(el) {
  el.style.display = "none";
}

function show(el) {
  el.style.display = "";
}

function enable(el) {
  el.disabled = false;
}

function disable(el) {
  el.disabled = true;
}

function submitForm(answer) {
  // Pretend it's hitting the network.
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (answer.toLowerCase() == "istanbul") {
        resolve();
      } else {
        reject(new Error("Good guess but a wrong answer. Try again!"));
      }
    }, 1500);
  });
}

let form = document.getElementById("form");
let textarea = document.getElementById("textarea");
let button = document.getElementById("button");
let loadingMessage = document.getElementById("loading");
let errorMessage = document.getElementById("error");
let successMessage = document.getElementById("success");
form.onsubmit = handleFormSubmit;
textarea.oninput = handleTextareaChange;
```

UI를 명령형으로 조작하는 방식은 독립적인 예제에서는 문제없이 동작하지만, 더 복잡한 시스템에서는 관리가 기하급수적으로 어려워집니다. 이와 같은 From들로 가득찬 페이지를 업데이트 한다고 상상해보세요. 새로운 UI 요소 혹은 새로운 인터렉션을 추가할 때 버그를 만들지 않기 위해서는, 기존 코드를 모두 조심스럽게 체크해야 합니다 (예를 들어, 무언가를 보여주고 숨기는 것을 까먹는다던가).

React는 이런 문제를 해결하기 위해서 만들어 졌습니다.

React에서는 UI를 직접 조작하지 않습니다. 컴포넌트를 직접적으로 활성화하고, 비활성화하고, 보여주거나 숨기지 않는다는 뜻입니다. 대신, 보여주고 싶은 것을 선언하면, React가 UI를 어떻게 업데이트할 것인지 찾아냅니다. 택시에 타서 정확히 어디서 돌아야하는지 말하는 것 대신에 어디에 가고 싶은지 말하는 것을 생각해보세요. 여러분을 그 곳까지 데려다 주는것은 기사님의 일이며, 생각지도 못한 지름길을 알고 있을 수도 있습니다!

## UI를 선언적으로 생각하기

위에서 Form을 명령형으로 구현하는 방법을 알아봤습니다. React에서 사고하는 법을 더 잘 이해하기 위해서, 이 UI를 아래에서 React로 재구현해 볼 것입니다.

- 컴포넌트의 다양한 시각 상태 **식별하기**
- 상태들이 변하는 트리거 **정의하기**
- `useState`로 상태를 메모리에 **기억하기**
- 필수적이지 않는 상태 값 **제거하기**
- 상태를 설정하기 위해 이벤트 핸들러와 **연결하기**

### Step 1: 컴포넌트의 다양한 시각 상태 식별하기

컴퓨터 공학에서, 여러 "상태들" 중 하나인 ["상태 머신"](https://en.wikipedia.org/wiki/Finite-state_machine)에 대해 들어봤을 겁니다. 만약 디자이너와 함께 일한다면, 다양한 "시각적 상태"에 데한 목업을 봤을겁니다. React는 디자인과 컴퓨터 공학의 교차점에 위치하기 때문에, 두 아이디어 모두 열감의 원천입니다.

첫째, 유저가 볼 수 있는 UI의 각기 다른 모든 "상태"들을 시각화 할 핗요가 있습니다.

- **Empty**: Form은 비활성화된 "Submit" 버튼을 가집니다.
- **Typing**: Form은 활성화된 "Submit" 버튼을 가집니다.
- **Submitting**: Form이 완전회 비활성화 됩니다. Spinner가 보입니다.
- **Success**: Form 대신에 “Thank you” 메시지가 보입니다.
- **Error**: 타이핑 상태와 동일하지만, 에러 메시지를 추가로 가집니다.

디자이너와 같이, 로직을 추가하기 전에 각기 다른 상태에대한 "목업"을 맏들고 싶을 수 있습니다. 예를 들어, 여기 Form의 시각적 일부에 대한 모의가 있습니다. 이 Mock은 `status`라 불리는 prop에 의해 컨트롤되며 `empty`가 기본값입니다.

```tsx
type Status = "empty" | "typing" | "submitting" | "success" | "error";

interface Props {
  status: Status;
}

export default function Form({ status = "empty" }: Props) {
  if (status === "success") {
    return <h1>That's right!</h1>;
  }
  return (
    <>
      <h2>City quiz</h2>
      <p>In which city is there a billboard that turns air into drinkable water?</p>
      <form>
        <textarea />
        <br />
        <button>Submit</button>
      </form>
    </>
  );
}
```

이름은 중요하지 않기 때문에, 저 prop을 원하는대로 불러도 괜찮습니다. `status = 'empty'`를 `status = 'success'`로 바꾸면 성공 메시지가 노출되는 것을 볼 수 있습니다. Mocking을 사용하면 로직을 연결하기 전에 UI를 빠르게 검증할 수 있습니다. 여기 동일한 컴포넌트의 보다 구체적은 프로토타입이 있으며, 여전히 `status` prop으로 "controlled" 됩니다.

```tsx
type Status = "empty" | "typing" | "submitting" | "success" | "error";

interface Props {
  status: Status;
}

export default function Form({ status = "empty" }: Props) {
  if (status === "success") {
    return <h1>That's right!</h1>;
  }
  return (
    <>
      <h2>City quiz</h2>
      <p>In which city is there a billboard that turns air into drinkable water?</p>
      <form>
        <textarea disabled={status === "submitting"} />
        <br />
        <button disabled={status === "empty" || status === "submitting"}>Submit</button>
        {status === "error" && <p className="Error">Good guess but a wrong answer. Try again!</p>}
      </form>
    </>
  );
}
```

### Step 2: 상태들이 변하는 트리고 정의하기

두 가지 방식의 입력에대한 반응으로 상태 업데이트를 트리거할 수 있습니다.

- **사람의 입력**: 버튼 클릭, 필드 타이핑, 링크 네비게이팅 등
- **컴퓨터의 입력**: 네트워크 응답 도착, 타임아웃 완료, 이미지 로딩 등

두 경우 모두, **UI를 업데이트하기 위해서 상태(State) 값을을 설정해야 합니다.** 개발중인 Form의 경우 몇가지 서로 다른 입력에 반응해 상태를 변경해야 합니다.

- **Input의 텍스트 변경**(사람)은 텍스트 박스가 비었는지 여부에 따라 상태를 *Empty*에서 *Typing*로 전환되거나 다시 돌아가야합니다.
- **제출 버튼 클릭**(사람)은 상태를 *Submitting*으로 전환해야 합니다.
- **네트워크 성공 응답**(컴퓨터)은 상태를 *Success*로 전환해야 합니다.
- **네트워크 실패 응답**(컴퓨터)은 에러 메시지와 맞게 상태를 *Error*로 전환해야 합니다.

> ### 주의
>
> 사람의 입력에는 종종 [이벤트 핸들러](https://react.dev/learn/responding-to-events)가 필요합니다!

이 흐름을 시각화하기 위해서, 각 상태를 이름이 적힌 동그라미로, 각 변경들을 두 상태 사이의 화살표로 종이에 그려봤습니다. 다양한 흐름을 이 방식으로 스케치하여, 구현 전에 버그들을 찾아낼 수 있습니다.

![Form States](https://react.dev/_next/image?url=%2Fimages%2Fdocs%2Fdiagrams%2Fresponding_to_input_flow.dark.png&w=1920&q=75)

###### 이미지 출처 (Reacting to Input with State - https://react.dev/learn/reacting-to-input-with-state)

### Step 3: useState로 상태를 메모리에 기억하기

다음 단계로 컴포넌트의 시각적 상태들을`useState`로 메모리에 기억할 필요가 있습니다. 단순성이 핵심입니다: 각 상태 조각들은 "움직이는 조각"이며, 여러분은 가능한 적은 "움직이는 조각"을 갖길 바랄겁니다. 복잡할수록 더 많은 버그가 발생할 수 있습니다.

반드시 있어야하는 상태부터 시작합니다. 예를 들어, Input의 `answer`를 저장해야하며, `error`에 (만약 존재한다면) 최신 에러를 저장해야 합니다.

```tsx
const [answer, setAnswer] = useState<string>("");
const [error, setError] = useState<Error | null>(null);
```

그리고 시각적 상태들중 어떤 것을 출력해야 하는지 기억할 상태 값이 필요합니다. 일반적으로 메모리에 기억하는 방식은 한가지 이상이기 떄문에 시험해볼 필요가 있습니다.

만약 최선의 방법을 즉각적으로 생각해내기 힘들다면, 가능한 모든 시각적 상태를 커버할 수 있는 충분한 상태를 추가하는 것으로 시작해보세요.

```tsx
const [isEmpty, setIsEmpty] = useState<boolean>(true);
const [isTyping, setIsTyping] = useState<boolean>(false);
const [isSubmitting, setIsSubmitting] = useState<boolean>(false);
const [isSuccess, setIsSuccess] = useState<boolean>(false);
const [isError, setIsError] = useState<boolean>(false);
```

첫 아이디어가 최선이 아닐 수 있지만, 상태를 리펙토링하는 것도 과정의 일부이기 떄문에 괜찮습니다!

### Step 4: 필수적이지 않는 상태 값 제거하기

상태 내용의 복제를 피하고 필수적인 것만 추적하고 싶을겁니다. 상태 구조를 리펙토링하는데 조금의 시간을 사용하면 컴포넌트를 이해하기 쉽게 만들고, 복제를 줄이며, 의도하지 않은 의미를 피할 수 있습니다. **메모리의 상태가 유저에게 보여주고 싶은 유효한 UI를 표현하지 않는 경우를 막는것이 목표입니다.** (예를 들어, 에러 메시지를 보여주고 않음과 동시에 Input을 비활성화 시키면, 유저의 오류를 수정할 수 없습니다.)

다음은 상태 변수에게 던질 수 있는 질문들 입니다.

- **이 상태가 모순을 일으키나요?** `isTyping`과 `isSubmitting`은 동시에 `true`가 될 수 없습니다. 모슨은 일반적으로 상태가 충분히 통제되지 않았음을 의미합니다. 두 불리언 값으로 4가지의 조합을 만들 수 있지만, 3개만 유효합니다. "불가능한" 상태를 제거하기 위해서, 이들을 3가지 값중 하나만 될 수 있는 `status`로 합칠 수 있습니다.
- **다른 상태 값에서 동일한 정보를 이미 사용할 수 있습니까?** 또 다른 모순은 `isEmpty`와 `isTyping`는 동시에 `true`가 될 수 없다는 것 입니다. 이들을 별개의 상태 변수로 만들면, 이들의 동기가 깨질 수 있는 위험이 생기며 버그가 발생할 수 있습니다. 다행히도, `isEmpty`를 제거하고 `answer.length === 0`로 대체할 수 있습니다.
- **다른 상태 변수의 반대 값으로 동일한 정보를 얻을 수 있습니가?** `error !== nul`를 사용하면 되기 때문에 `isError` 는 필요하지 않습니다.

```tsx
type Status = "typing" | "submitting" | "success";

const [answer, setAnswer] = useState<string>("");
const [error, setError] = useState<boolean>(null);
const [status, setStatus] = useState<Status>("typing");
```

이들은 전부 필수적입니다. 이들중 한개라도 빠진다면 기능이 제대로 동작하지 않습니다.

### Step 5 : 상태를 설정하기 위해 이벤트 핸들러와 연결하기

마지막으로, 상태를 업데이트하는 이벤트 핸들러를 만듭니다. 아래는 모든 이벤트 핸들러가 연결된 최종 Form입니다.

```tsx
import { useState } from "react";

type Status = "typing" | "submitting" | "success";

export default function Form() {
  const [answer, setAnswer] = useState<string>("");
  const [error, setError] = useState<Error>(null);
  const [status, setStatus] = useState<Status>("typing");

  if (status === "success") {
    return <h1>That's right!</h1>;
  }

  async function handleSubmit(e) {
    e.preventDefault();
    setStatus("submitting");
    try {
      await submitForm(answer);
      setStatus("success");
    } catch (err) {
      setStatus("typing");
      setError(err);
    }
  }

  function handleTextareaChange(e) {
    setAnswer(e.target.value);
  }

  return (
    <>
      <h2>City quiz</h2>
      <p>In which city is there a billboard that turns air into drinkable water?</p>
      <form onSubmit={handleSubmit}>
        <textarea value={answer} onChange={handleTextareaChange} disabled={status === "submitting"} />
        <br />
        <button disabled={answer.length === 0 || status === "submitting"}>Submit</button>
        {error !== null && <p className="Error">{error.message}</p>}
      </form>
    </>
  );
}

function submitForm(answer: string) {
  // Pretend it's hitting the network.
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      let shouldError = answer.toLowerCase() !== "lima";
      if (shouldError) {
        reject(new Error("Good guess but a wrong answer. Try again!"));
      } else {
        resolve();
      }
    }, 1500);
  });
}
```

비록 기본의 원본 예제보다 코드가 길어졌지만, 더욱 단단합니다. 상태 변경으로 모든 인터렉션들을 표현하면 나중에 기존의 것들을 깨뜨리지 않고 새로운 시각적 상태를 추가할 수 있습니다. 또한 인터렉션 로직을 수정하지 않고 각 생태에 따라 무엇이 보여져야 하는지를 수정할 수 있습니다.

## 요약

- 선언적 프로그래밍은 UI를 세밀하게 조작하는 대신 각 시각적 상태에 대한 UI를 묘사하는 것을 의미합니다.
- 컴포넌트를 개발 할 때:
  - 다양한 시각 상태 **식별하기**
  - 상태 변경을 위한 사람과 컴퓨터의 트리거 **정의하기**
  - `useState`로 상태 **설계하기**
  - 버그와 모순을 피하기 위해 필수적이지 않는 상태 **제거하기**
  - 상태와 이벤트 핸들러 **연결하기**
