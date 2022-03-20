# Getting Started

`yarn`을 사용해 Jest 설치하기:

```zsh
yarn add --dev jest
```

혹은 `npm` 사용:

```zsh
npm install --save-dev jest
```

주의: Jest 문서는 `yarn` 명령어를 사용지만, `npm` 도 작동합니다. `yarn`과 `npm` 명령어는 [yarn 문서](https://classic.yarnpkg.com/en/docs/migrating-from-npm#toc-cli-commands-comparison)에서 비교할 수 있습니다.

두 수를 더하는 예제 함수에 대한 테스트를 작성하는 것으로 시작해 봅시다. 우선, `sum.js` 파일을 생성합니다:

```javascript
function sum(a, b) {
  return a + b;
}

module.exports = sum;
```

그리고 `sum.test.js` 파일을 생성합니다. 여기에 시렞 테스트 코드를 담습니다:

```javascript
const sum = require("./sum");

test("adds i + 2 to equal 3", () => {
  expect(sum(1, 2)).toBe(3);
});
```

다음 section을 `package.json`에 추가합니다:

```json
{
  "script": {
    "test": "jset"
  }
}
```

마지막으로, `yarn test` 혹은 `npm run test`를 실행하면 Jest가 메시지를 출력할 것입니다:

```zsh
PASS  ./sum.test.js
✓ adds 1 + 2 to equal 3 (5ms)
```

#### Jest를 사용해 첫 테스트를 작성하는데 성공했습니다!

이 테스트에서 두 수가 동일한지 테스트하기 위해서 `expect`와 `toBe`를 사용했습니다. Jest가 테스트할 수 있는 다른 것들에 대해서도 알고싶다면, [Matchers 사용법](https://jestjs.io/docs/using-matchers)를 살펴보세요.

## 커멘드 라인으로 실행

CLI에서 Jest를 다양한 옵션 값과 함께 직접 실행할 수 있습니다 (`yarn global add jest` 혹은 `npm install jest --global`을 통해 Jest가 `PATH` 내부에서 global하게 사용할 수 있다면).

다음을 실행하면 Jest는 `my-test` 파일로 테스트를 실행합니다. 또한 `config.json`를 설정 파일로 사용하고 실행이 끝나면 네이티브 OS notification을 보여줍니다.

```zsh
jest my-test --notify --config=config.json
```

`jest`를 커멘드 라인으로 실행시키는 방법에 대해 더 자세히 알고싶다면, [Jest CLI Options](https://jestjs.io/docs/cli)페이지를 살펴보세요.

## 추가 설정

### 기본 설정 파일 생성하기

프로젝트를 기반으로 Jest는 몇가지 질문을 한 후에 각 옵션에 대한 짧은 설명이 담긴 기본 설정 파일을 생성합니다.

```zsh
jest --init
```

### Babel 사용하기

[바벨](https://babeljs.io/)을 사용하기 위해서는 `yarn`을 통해 필요한 dependency들을 설치해야 합니다.

```zsh
yarn add --dev babel-jest @babel/core @babel/preset-env
```

프로젝트 root에 `babel.config.js`파일을 생성하여 현재 Node 버전을 Bebel에 설정합니다.

```javascript
module.exports = {
  presets: [["@babel/preset-env", { targets: { node: "current" } }]],
};
```

Babel을 위한 이상적인 설정은 프로젝트마다 다릅니다. [Babel 문서](https://babeljs.io/docs/en/)에서 더 자세한 사항을 확인하세요.

Jest는 별도의 설정이 없다면 `process.env.HODE_ENV`를 `test`로 할당합니다. 이것을 이용하면 Jest에 필요한 컴파일을 조건부로 설정할 수 있습니다.

```javascript
module.exports = (api) => {
  const isTest = api.env("test");

  return {
    // ...
  };
};
```

> `babel-jest`는 Jest를 설치하면 자동적으로 설치되고, 바벨 설정이 존재하면 프로젝트의 파일을 자동으로 변환합니다. 이것은 `transform` 옵션을 명시적으로 리셋해서 피할 수 있습니다.

```javascript
module.exports = {
  tranform: {},
};
```

### 웹팩 사용하기

Jest는 [웹팩](https://webpack.js.org/)을 사용해 assets, styles, 컴파일을 관리하는 프로젝트에서 사용할 수 있습니다. 웹팩은 다른 툴에 비해 몇 가지 특별한 문제들을 제공합니다. 시작하려면 [웹팩 가이드](https://jestjs.io/docs/webpack)를 확인하세요.

### parcel 사용하기

Jest는 [parcel-bundler](https://parceljs.org/)를 사용해 assets, styles, 컴파일을 관리하는 프로젝트에서 사용할 수 있습니다. parcel은 설정이 필요하지 않습니다. 시작하려면 [공식 문서](https://parceljs.org/docs/)를 확인하세요.

## Typescript 사용하기

### Babel을 통해 Typescript 사용하기

앞서 설명한 **Babel 사용하기**룰 먼저 수행합니다. 그리고, `yarn`을 통해 `@babel/preset-typescript`를 설치합니다.

```zsh
yarn add --dev @babel/preset-typescript
```

그리고 `babel.config.js`의 presets에 `@babel/preset-typescript`를 추가합니다.

```javascript
module.exports = {
  preset: [["@babel/preset-env", { targets: { node: "current" } }], "@babel/preset-typescript"],
};
```

그러나 Babel과 함께 TypeScript를 사용할 때 몇 가지 [주의 사항](https://babeljs.io/docs/en/babel-plugin-transform-typescript#caveats)이 있습니다. Babel의 Typescript 지원은 변환이 전부이기 떄문에, Jest는 테스트가 실행될 때 타입 체크를 진행할 수 없습니다. 만약 타입 체크를 원한다면 [ts-jest](https://jestjs.io/docs/getting-started#:~:text=you%20can%20use-,ts%2Djest,-instead%2C%20or%20just)를 대신 사용하거나, 타입스크립트 컴파일러인 [tsc](https://www.typescriptlang.org/docs/handbook/compiler-options.html)를 별도로(혹은 빌드 과정의 일부로) 실행해야 합니다

### ts-jest를 통해 TYpescript 사용하기

[ts-jest](https://github.com/kulshekhar/ts-jest)는 타입스크립트 전처리기이며, 소스 맵을 지원하여 Jest에서 타입 스크립트로 작성된 테스트를 사용할 수 있게 해줍니다.

사용중인 Jest 버전을 위한 `@types/jest` 모듈을 사용하면, 테스트를 전부 타입 스크립트로 작성하는데 도움이 됩니다.

```zsh
yarn add --dev @types/jest
```
