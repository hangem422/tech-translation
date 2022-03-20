# Snapshot Testing

스냅샷 테스트들은 UI가 의도하지 않게 변경되는지 확인하고 싶을 때 매우 유용한 툴입니다.

전현적으로 스냅샷 테스트 케이스는 UI 컴포넌트를 랜더하고, 스냅샷을 촬영하고, 테스트 근처에 저장된 레퍼런스 스냅샷과 비교하는 과정을 갖습니다. 두 스냅샷이 다르거나 의도한 변화가 일어나지 않으면 테스트는 실패합니다. 그렇지 않으면 레퍼런스 스냅샷이 새로운 버전의 UI 컴포넌트로 업데이트돼야 합니다.

## Snapshot Testing with Jest

React 컴포넌트 테스트에서도 비슷한 접근이 가능합니다. 전체 앱을 빌드하는데 필요한 그래픽 UI를 랜더하는 것 대신에, React 트리에 필요한 serializable value를 빠르게 생산하여 렌더를 테스트 할 수 있습니다. [Link 컴포넌트](https://github.com/facebook/jest/blob/main/examples/snapshot/Link.js)를 위한 [예제 테스트](https://github.com/facebook/jest/blob/main/examples/snapshot/__tests__/link.test.js)를 확인해보세요.

```jsx
import React from "react";
import renderer from "react-test-renderer";
import Link from "../Link";

it("render correctly", () => {
  const tree = renderer.create(<Link page="http://www.facebook.com">Facebook</Link>).toJson();
  expect(tree).toMatchSnapshot();
});
```

첫번째로 test가 실행되면, Jest는 다음과 같은 [스탭샷 파일](https://github.com/facebook/jest/blob/main/examples/snapshot/__tests__/__snapshots__/link.test.js.snap)을 생성합니다.

```jsx
exports[`renders correctly 1`] = `
<a
  className="normal"
  href="http://www.facebook.com"
  onMouseEnter={[Function]}
  onMouseLeave={[Function]}
>
  Facebook
</a>
`;
```

스냅샷 아티팩트는 코드 변경과 함께 커밋되고 코드 검토 프로세스의 과정으로 검토됩니다. Jest는 [prettu-format](https://github.com/facebook/jest/tree/main/packages/pretty-format)을 사용해 코드를 리뷰하는 동안 스냅샷을 사람이 보기 좋은 형태로 변경합니다. 테스트가 실행된 직후, Jest는 렌더된 결과를 이전 스냅샷과 비교하고, 만약 일치한다면 테스트가 통과됩니다. 만약 불일치 하면 코드에 버그가 있거나 구현이 변경됐기 때문에 스냅샷이 업데이트 돼야하는 상황입니다.

> 주의: 스냅샷은 데이터를 렌더한 스코프 내부에서만 유효합니다. - 예제에서 `<Link />` 컴포넌트는 `page`를 prop으로 받습니다. 만약 다른 파일에서 `<Link />` 컴포넌트에 prop을 생략한다해도, 테스트는 `<Link />` 컴포넌트의 사용법을 모르며 스냅샷의 스코프는 `Link.js`로 한정되어 있기 떄문에 테스트는 여전히 통과하빈다. 동일한 컴포넌트를 다른 props로 렌더하더라도 각 테스트는 서로 모르기 때문에 다른 스냅샷은 첫 스냅샷에 영향을 끼치지 않습니다.

스냅샷 테스트가 어떻게 동작하는지 좀 더 많은 정보가 필요하거나 왜 이것을 만들었는지 궁금하다면 [release blog post](https://jestjs.io/blog/2016/07/27/jest-14)를 참고하세요. 스냅샷 테스트를 좀 더 잘 사용하고 싶다면 이 [블로그 포스트](http://benmccormick.org/2016/09/19/testing-with-jest-snapshots-first-impressions/)를 추천합니다. [egghead video](https://jestjs.io/docs/snapshot-testing) 또한 추천합니다.

### Updating Snapshots

버그가 보고된 후 스냅샷 테스트가 실패한 지점을 발견하는 것은 간단합니다. 버그가 발생하면 이슈를 해결하고 스냅샷 테스트가 통과하는지 다시 확인하세요. 이제 스냅샷 테스트가 내부 구현 변경때문에 일어났을 때에 대해 이야기 해 봅시다.

만약 내부적으로 Link 컴포넌트의 주소가 변경딘 경우를 예로 들겠습니다.

```jsx
it("renders correctly", () => {
  const tree = renderer.create(<Link page="http://www.instagram.com">Instagram</Link>).toJSON();
  expect(tree).toMatchSnapshot();
});
```

컴포넌트가 다른 주소를 바라보게 했기 떄문에, 이 컴포넌트의 스냅샷은 변경될 것이라고 예상할 수 있습니다. 스냅샷 테스트는 이 테스트의 스냅샷 아티펙트와 업데이트된 컴포넌트의 스냅샷이 다르기 떄문에 실패할 것 입니다.

이를 해결하기 위해서, 우리는 스냅샷 아티펫트를 업데이트할 필요가 있습니다. 스냅샷을 재 생성하는 Jest 플래를 사용할 수 있습니다.

```zsh
jest --updateSnapshot
```

위 명령어로 변경사항을 적용해 보세요. 취향에따라 `-u` 단일 단어 플래그로 대체할 수도 있습니다. 이 명령어는 실패한 모든 테스트의 snapshot 아티펙트를 다시 생성합니다. 만약 스냅샷 테스트 실패가 추가적으로 발생한다면 다른 버그일 것이며, 버그록인한 스냅샷을 피하기 위해 재생성 전에 수정해야 합니다.

스냅샷의 재생성을 특정 패턴으로 한정하고 싶다면 `--testNamePattern=<regex>` 플래그를 추가할 수 있습니다.

### Interactive Snapshot Mode

watch mod에서는 실패한 스냅샷을 인터렉티브하게 업데이트 할 수 있습니다.

Ineractive 스냅샷 모드를 실행하면, Jest 테스트의 실패한 스냅샷을 한 번에 한 테스트씩 단계별로 안내하고 실패한 결과를 검토할 기회를 제공합니다.

완료하고나면 Jest는 watch 모드로 돌아가기 전에 summary를 제공합니다.

### Inline Smapshots

인라인 스냅샷은 스냅샷 결과가 자동으로 소스 코드 내부에 다시 기록되는 것을 제외하고 external 스냅샷과 동일하게 동작합니다. 이것은 자동으로 스냅샷이 옳바르게 생성됐는지 확인하는 과정을 외부 파일이 아닌 내부에서 확인할 수 있는 이점이 있다는 것을 의미합니다.

#### Example

`toMatchInlineSnapshot()`를 인자 없이 사용하여 테스트를 작성합니다.

```javascript
it("renders correctly", () => {
  const tree = renderer.create(<Link page="https://example.com">Example Site</Link>).toJSON();
  expect(tree).toMatchInlineSnapshot();
});
```

다음으로 Jest를 실행하면 , tree가 평가되고 스냅샷은 `toMatchInlineSnapshot`의 인자 값으로 기록됩니다.

```javascript
it("renders correctly", () => {
  const tree = renderer.create(<Link page="https://example.com">Example Site</Link>).toJSON();
  expect(tree).toMatchInlineSnapshot(`
<a
  className="normal"
  href="https://example.com"
  onMouseEnter={[Function]}
  onMouseLeave={[Function]}
>
  Example Site
</a>
`);
});
```

이게 전부입니다! 심지어 `--watch` 모드에서 `--updateSnapshot` 혹은 `u` 키를 이용해 스냅샷을 업데이트 하는 것도 가능합니다.

기본적으로 Jest는 소스코드에 스냅샷을 기록합니다. 하지만, [prettier](https://www.npmjs.com/package/prettier)을 프로젝트에서 사용하고 있다면, Jest는 이것을 인지하고 prettier에게 이 작업을 위임합니다. (설정값을 준수하여)

### Property Matchers

스냅샷은 애플리케이션 내부에서 예상하지 못한 인터페이스 변화를 인지하는데 환상적인 툴입니다. 그인터페이스가 API 결과, UI, logs, 혹은 에러 메시지이 경우에도 말입니다. 전략적이고 효과적인 테스트를 위해서 알아야한 best-parctice와 가이드라인이 존재합니다.

#### 1. 스냅샷을 코드로 취급학세요

일반적인 코드의 리뷰 과정의 일부로서 스냅샷을 커밋하고 리뷰하세요 이 말은 스냅샷을 프로젝트의 다른 테스트 혹은 코드와 동일하게 취급하라는 말입니다.

스냅샷 일기에 충분히 짧고 집중할 수 있는지 확인하고, 툴을 사용하여 스타일 컨벤션을 유지하게요.

이전에 말씀드린 것처럼, Jest는 `pretty-format`을 사용해 사람이 읽기 쉬우나, `eslint-plugin-jest`의 `no-large-snpashots` 옵션 혹은 `snapshot-diff`의 컴포넌트 스냅샷 비교 기능과 같은 유용한 툴을 추가하여 짧고 집중력 있는 스냅샷을 생상하세요.

Pull Request의 스냅샷을 리뷰하기 쉽도록 하는 것이 목적이이며, 실패의 원인을 시험하는 것 대신에 스냅샷을 자주 변경하는 습관을 막아야합니다.

#### 2. 테스트는 결정적이어야 합니다.

테스트는 결정적이어야 합니다. 변경되지 않은 한 컴포넌트에서 같은 테스트를 여러번 시행하면 매번 같은 결과를 생성합니다. 특정 플랫폼의 스냅샷을 포함하거나 결정적이지 않은 데이터를 포함하는 스냅샹을 생산하지 않은 책임이 있습니다.

예를 들어, 만약 시계 컴포넌트에서 `Date.now()`를 사용한다면, 스냅샷은 매 테스트마다 다른컴포넌트를 생산할 것 입니다. 이런 경우 `Date.now()` 메서드를 고정된 값을 반환하도록 mock 할 수 있습니다.

```javascript
Date.now = jest.fn(() => 1482363367071);
```

이제 , 모든 테스트 실행에서 `Date.now`는 `1482363367071`를 고정적으로 반환합니다. 이제 테스트를 언제 시행하든 관계 없이 스냅샷은 같은 결과를 생성합니다.

#### 3. 설명적인 스냅샷 이름을 생성하세요.

언제나 설명적인 이름을 정하려고 노력해야 합니다. 가장 최상의 이름은 스냅샷 내용을 설명하는 이름입니다. 이것은 리뷰어들에게 리뷰를 하는 동안 스냅샷을 검증하기 쉽게 도와주며, 업데이트 전의 스냅샷의 정확한 역할을 알 수 있게 해줍니다.

```javascript
exports[`<UserName /> should handle some test case`] = `null`;

exports[`<UserName /> should handle some other test case`] = `
<div>
  Alan Turing
</div>
`;
```

```javascript
exports[`<UserName /> should render null`] = `null`;

exports[`<UserName /> should render Alan Turing`] = `
<div>
  Alan Turing
</div>
`;
```

## 자주 묻는 질문

### CI 시스템에서 스냅샷이 자동으로 작성됩니까?

아닙니다. Jest 20에서는 `--updateSnapshot` 을 사용하지 않으면 CI 시스템에서 Jest는 스냅샷을 자동으로 기록하지 않습니다. 모든 스냅샷은 CI에서 실행되는 코드의 일부이며, 새 스냅샷이 자동으로 전달될 때 Ci 시세템에서 테스트가 통과되면 안됩니다. 모든 스냅샷은 항상 커밋되고 버진이 관리되어야 합니다.

### 스냅샷 파일은 커밋되어야 하나요?

네, 모든 스냅샷 파일은 해당 모듈 및 테스트와 함께 커밋돼야 합니다. 스냅샷들은 테스트의 일부로 간주돼애 하며, Jest의 다른 assertion 값들과 유사합니다. 사실, 스냅샷은 해당 시점의 소스 모듈의 상태를 나타냅니다. 이 방식대로라면, 소스 모듈이 변경되면, Jest는 이전 버전과 무엇이 달라졌는지 말 할 수 있습니다. 그리고 코드 리뷰중에 리뷰어들이 변화를 좀 더 잘 습득할 수 있는 추가적인 컨텍스트들도 제공할 수 있습니다.

### 스냅샷은 리엑트 컴포넌트만을 위해 사용되나요?

React와 React Native 컴포넌트들은 스냅샷의 훌륭한 유즈 케이스입니다. 그러나, 스냅샷은 serializable 값을 캡쳐할 수 있고, 테스트의 목적이 올바른지 검사하기 위한 곳이라면 언제든 사용될 수 있습니다. Jest 레파지토리는 Jest 자체의 출력을 테스트하는 많은 예제가 포함되어 있으며, Jest의 assertopm 라이브러리 출력 뿐만 아니라 Jest 코드베이스 다양한 부분의 로그가 있습니다. Jest 레파지토리의 [snpashotting CLI output](https://github.com/facebook/jest/blob/main/e2e/__tests__/console.test.ts) 예제를 살펴보세요.

### 스냅샷 테스트와 Visual Regression 테스트의 차이점이 무엇인가요?

두 방법은 UI를 테스트하는 별개의 방법이며 다른 목적을 가지고 있습니다. Visual regression ㅌ테스트 툴은 웹 페이지를 스크린샷하고 이미지의 픽셀을 비교합니다. 스냅샷 테스트의 값은 직렬화되고, 텍스트 파일로 저장되며, diff 알고리즘으로 비교됩니다. 여기에는 고민해야 할 다양한 절충안들이 있으며 [Jest 블로그](https://jestjs.io/blog/2016/07/27/jest-14#why-snapshot-testing)에 스냄샷 테스트가 구축된 이유를 적어놨습니다.

### 스냅샷 테스트는 유닛 테스트를 대체할 수 있나요?

스냅샷 테스트는 Jest와 함계 제공되는 20개 이상의 aseertion중 하나입니다. 스냅샷 테스트의 목적은 다른 유닛 테스트를 대체하기 위한 것이 아니며, 추가적인 가치를 제공하고 테스트 고통을 줄여주기 위한 것입니다. 일부 시나리오에서 스냅샷 테스트는 특정 기능에대한 유닛 테스트의 필요성을 없애줄 수 있지만, 함계 적용할 수 도 있습니다.

### 생성되는 파일의 크기와 속도를 고려해서 스냅샷 테스트의 성능은 어떠한가요?

Jest는 성능을 고려하고 작성되었으며 스냅샷 테스트도 예외는 아닙니다. 슨배샷이 파일로 저장되기 때문에 이 테스트 방법은 빠르고 안정적입니다. Jest는 `toMatchSnapshot` matcher가 실행된 파일마다 한 개의 파일을 생성합니다. 스냅샷 파일의 사이즈는 상당히 작습니다. 참고로 Jest 코드베이스 자체이 있느 ㄴ모든 스냅샷 파일의 크기는 300KB 미만입니다.

### 스냅샷 파일의 충돌은 어떻게 해결해야 하나요?

스냅샷 파일은 항상 관련된 모듈의 현재 상태를 나타내야 합니다. 그렇기에, 만약 두개의 브랜치가 합쳐지고 스냅샷 파일에서 충돌이 발생하면, 수작업으로 충돌을 해결하거나 jest를 실행하여 스냅샷을 업데이트하고 결과를 반영하면 됩니다.

### 스냅샷 테스트를 TDD에 적용하는 것이 가능한가요?

수작업으로 스냅샷 파일을 기록하는게 가능할지라도, 일반적으로 쉽지 않습니다. 스냅샷은 처음부터 코드를 설계하기 위한 가이드를 제공한다기 보다는 관련된 모듈의 결과가 변경되었는지 파악하는데 도움이 됩니다.

### 코드 검사가 스냅샷 테스트와 함계 작동하나요?

네, 다른 테스트도 마찬가지입니다.
