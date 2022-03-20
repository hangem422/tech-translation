# Testing React Apps

Facebook에서는 Resact 애플리케이션 테스트를 위해 Jest를 사용합니다.

## Setup

### Setup with Create React App

만약 새로운 리엑트 프로젝트를 시작하려 한다면, [Create React App](https://jestjs.io/docs/tutorial-react)을 추천합니다. 여기에는 Jest가 미리 준비돼있습니다. 스냅샷 랜더링을 위해서 `react-test-renderer`만 추가하면 됩니다.

싫행

```zsh
yarn add --dev react-test-renderer
```

### Setup without Create React App

만약 이미 애플리케이션이 존재한다면 몇가지 패키지를 설치하여 사용할 수 있습니다. 테스트 환경의 코드를 변환하기 위해 `babel-jest` 패키지와 `react` 바벨 프리셋을 사용합니다.

실행

```yarn
yarn add --dev jest babel-jest @babel/preset-env @babel/preset-react react-test-renderer
```

`package.json`의 모양은 아래와 같습니다 (`<current-version>`은 패키지의 최신 버전을 의미하빈다). script와 jest 설정을 추가해주세요.

#### package.json

```json
{
  "dependencies": {
    "react": "<current-version>",
    "react-dom": "<current-version>"
  },
  "devDependencies": {
    "@babel/preset-env": "<current-version>",
    "@babel/preset-react": "<current-version>",
    "babel-jest": "<current-version>",
    "jest": "<current-version>",
    "react-test-renderer": "<current-version>"
  },
  "scripts": {
    "test": "jest"
  }
}
```

#### babel.config.js

```javascript
module.exports = {
  presets: ["@babel/preset-env", "@babel/preset-react"],
};
```

## Snapshot Testing

```jsx
import React, { useState } from "react";

const STATUS = {
  HOVERED: "hovered",
  NORMAL: "normal",
};

const Link = ({ page, children }) => {
  const [status, setStatus] = useState(STATUS.NORMAL);

  const onMouseEnter = () => {
    setStatus(STATUS.HOVERED);
  };

  const onMouseLeave = () => {
    setStatus(STATUS.NORMAL);
  };

  return (
    <a className={status} href={page || "#"} onMouseEnter={onMouseEnter} onMouseLeave={onMouseLeave}>
      {children}
    </a>
  );
};

export default Link;
```

> 주의: 예제는 함수형 컴포넌트를 사용합니다. 하지만 클래스 컴포넌트도 같은 방법으로 테스트 할 수 있습니다. [React: Function and Class](https://reactjs.org/docs/components-and-props.html#function-and-class-components)를 참고하세요. 클래스 컴포넌트를 사용하면 메서드를 직접 테스트하는 방식이 아니라, props를 테스트하는데 Jest가 사용되길 바랍니다.

이제 컴포넌트의 상호작용과 랜더링된 결과를 캡쳐하고 스냅샷 파일을 생성하기 위해 React의 test renderer와 Jest의 스냅샷 기능을 사용해봅시다.

```javascript
import React from "react";
import renderer from "react-test-renderer";
import Link from "../Link";

test("Link changes the class when hovered", () => {
  const component = renderer.create(<Link page="http://www.facebook.com">Facebook</Link>);
  let tree = component.toJson();
  expect(tree).toMatchSnapshot();

  tree.props.onMouseEnter();
  tree = component.toJson();
  expect(tree).toMatchSnapshot();

  tree.props.onMouseLeave();
  tree = component.toJson();
  expect(tree).toMatchSnapshot();
});
```

`yarn test` 혹은 `jest`를 실행하면 다음과 같은 파일이 생성됩니다.

```javascript
exports[`Link changes the class when hovered 1`] = `
<a
  className="normal"
  href="http://www.facebook.com"
  onMouseEnter={[Function]}
  onMouseLeave={[Function]}>
  Facebook
</a>
`;

exports[`Link changes the class when hovered 2`] = `
<a
  className="hovered"
  href="http://www.facebook.com"
  onMouseEnter={[Function]}
  onMouseLeave={[Function]}>
  Facebook
</a>
`;

exports[`Link changes the class when hovered 3`] = `
<a
  className="normal"
  href="http://www.facebook.com"
  onMouseEnter={[Function]}
  onMouseLeave={[Function]}>
  Facebook
</a>
`;
```

다음 테스트 실행시 랜더링 결과는 이전에 생성된 스냅샷과 비교됩니다. 스냅샷은 코드 변경과 함께 커밋돼야 합니다. 스냅샷 테스트가 실패하면, 이것의 의도된 변화인지 아닌지를 조사해야 합니다. 만약 예상된 변화였다면 기존 스냅샷을 덮어씌우기 위해 `jest -u`를 실행해야합니다.

[example/snapshot](https://github.com/facebook/jest/tree/main/examples/snapshot)에서 예제를 확인할 수 있습니다.

### Snapshot Testing with Mocks, Enzyme and React 16

Enzyme와 React 16+를 사용해서 스냅샷 테스트를 진행하는 경우 주의사항이 있습니다. 만약 아래와같이 module을 mocking해서 사용한다면

```javascript
jest.mock("../SomeDirectory/SomeComponent", () => "SomeComponent");
```

다음과 같은 경고를 보게 됩니다.

```zsh
Warning: <SomeComponent /> is using uppercase HTML. Always use lowercase HTML tags in React.

# Or:
Warning: The tag <SomeComponent> is unrecognized in this browser. If you meant to render a React component, start its name with an uppercase letter.
```

React 16에서는 엘리먼트의 타입을 검사하고, mocking 된 모듈이 이 검사를 통과하지 못하기 떄문에 경고를 출력합니다. 다음과 같은 대안들이 있습니다.

1. 텍스트를 랜더링합니다. 이 방법은 직관적이지만 prop의 전송을 스냅샷에서 볼 수 없습니다.

```javascript
jest.mock("./SomeComponent", () => () => "SomeComponent");
```

2. 커스텀 엘리먼트를 랜더합니다. DOM은 "custom elements"를 체크하지 않기 떄문에 경고가 발생하지 않습니다. 이들인 이름은 소문자와 -를 갖습니다.

```javascript
jest.mock("./Widget", () => () => <mock-widget />);
```

3. `react-test-renderer`를 사용합니다. 테스트 랜더러는 엘리먼트의 타입을 신경쓰지 않으며 `"SomeComponent`와 같은 것을 적용할 수 있습니다. 테스트 렌더러를 사용해 스냅샷을 체크할 수 있으며, 체크 행위는 부분적으로 Enzyme를 사용합니다.
4. 경고를 해제합니다. (jest 설정 파일에서 수행해야 합니다)

```javascript
jest.mock("fbjs/lib/warning", () => require("fbjs/lib/emptyFunction"));
```

이 방법은 유용한 경고도 손실될 수 있으므로 일반적으로 선택하지 않아야 합니다. 그러나, 예를 들어, 리엑트 네이티브 컴포넌트들을 테스트 할 때 리엑트 네이티브 태그를 DOM에 랜더링하는 경우처럼 많은 경고가 무의미할 때 사용할 수 있습니다. 또 다른 옵션은 `console.warn`을 재구성해서 특정 경고를 표시하지 않게 하는 것 입니다.

## Dom Testing

만약 랜더링된 컴포넌트를 조작하여 테스트하고 싶다면 [react-testing-library](https://github.com/kentcdodds/react-testing-library), [Enzyme](http://airbnb.io/enzyme/), 혹은 리엑트 [TestUtils](https://reactjs.org/docs/test-utils.html)를 사용할 수 있습니다.

### react-testing-library

`yarn add --dev @testing-library/react`로 react-testing-library를 사용할 수 있습니다.

두 라벨을 바꾸는 체크박스를 구현해 봅시다.

```jsx
import React, { useState } from "react";

const CheckmocWithLabel = ({ labelOn, labelOff }) => {
  const [isChecked, setIsChecked] = useState(false);

  const onChange = () => {
    setIsChecked(!isChecked);
  };

  return (
    <label>
      <input type="checkbox" checked={isChecked} onChange={onChange} />
      {isChecked ? labelOn : labelOff}
    </label>
  );
};
```

```javascript
import React from "react";
import { cleanup, fireEvent, render } from "@testing-library/react";
import CheckboxWithLabel from "../CheckboxWithLabel";

// 주의: cleanup afterEach는 @testing-library/react@9.0.0 이상부터 자동으로 수행됩니다.
// 테스트가 끝난 후 언마운트하고 DOm을 초기화합니다.
afterEach(cleanup);

it("CheckboxWithLabel changes teh text after click", () => {
  const { queryByLabelText, getByLabelText } = render(<CheckboxWithLabel labelOn="On" labelOff="Off" />);

  expect(queryByLabelText(/off/i)).toBeTruthy();

  fireEvent.click(getByLabelText(/off/i));

  expect(queryByLabelText(/on/i)).toBeTruthy();
});
```

### Enzyme

Enzyme를 사용하기 위해서는 `yarn add -dev enzyme`를 실행해야 합니다. 리엑트 버전이 15.5.0 이하라면 `react-addons-test-utils`도 함께 설치해야 합니다.

```javascript
import React from "react";
import { shallow } from "enzyme";
import CheckboxWithLabel from "../CheckboxWithLabel";

test("CheckboxWithLabel changes the text after click", () => {
  // Render a checkbox with label in the document
  const checkbox = shallow(<CheckboxWithLabel labelOn="On" labelOff="Off" />);

  expect(checkbox.text()).toEqual("Off");

  checkbox.find("input").simulate("change");

  expect(checkbox.text()).toEqual("On");
});
```
