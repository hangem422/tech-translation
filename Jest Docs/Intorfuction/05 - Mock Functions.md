# Mock Functions

Mock 함수를 사용하면 함수의 실제 실행을 지우고, 함수 호출을 캡처하고(호출에 전달된 매개변수까지), `new`로 호출된 생성자 함수의 인스턴스를 캡쳐하고, 반환값의 테스트 타임 설정을 허용하여 코드간의 연결을 테스트할 수 있습니다.

두가지 방법의 Mock 함수가 있습니다: 테스트 코드 내부에서 사용할 mock 함수를 생성하거나, `manual mock`을 작성하여 의존성 모듈을 오버라이드 할 수 있습니다.

## mock 함수 사용하기

배열의 각 아이템에 대해서 callback을 실행하는 `forEach` 함수의 실행을 테스트 한다고 상상해봅시다.

```javascript
function forEach(items, callback) {
  for (let index = 0; index < items.length; index++) {
    callback(items[index]);
  }
}
```

함수를 테스트하기 위해서 우리는 mock 함수를 사용할 수 있으며, callback이 예상대로 호출되고 있는지 검증할 수 있습니다.

```javascript
const mockCallback = jest.fn((x) => 42 + x);
forEach([0, 1], mockCallback);

// mock 함수는 2번 호출됩니다.
expect(mockCallback.mock.calls.length).toBe(2);

// 첫번째 호출의 첫번째 인자는 0입니다.
expect(mockCallback.mock.calls[0][0].toBe(0));

// 두번째 호출의 첫번쨰 인자는 1입니다.
expect(mockCallback.mock.calls[1][0].toBe(1));

// 첫번째 호출의 반환값은 42입니다.
expect(mockCallback.mock.results[0].value).toBe(42);
```

## mock 프로퍼티

모든 mock 함수들은 함수가 어떻게 호출됐고 무엇을 반환했는지를 가지는 특별한 `.mock` 프로퍼티를 갖습니다. 또한 `.mock` 프로퍼티는 호출마다 `this` 값을 추적할 수 있습니다.

```javascript
const myMock = jest.fn();

const a = new myMock();
const b = {};
const bound = myMock.bind(b);
bound();

console.log(myMock.mock.instances);
// > [ <a>, <b> ]
```

이런 mock의 맴버들은 함수들이 어떻게 호출, 인스턴스화 됐는지 혹은 무엇을 반환했는지를 테스트할 때 매우 유용합니다.

```javascript
// 함수는 한번 호출됐습니다.
expect(someMockFunction.mock.calls.length).toBe(1);

// 첫 호출의 첫번째 인자는 'first arg' 입니다.
expect(someMockFunction.mock.calls[0][0]).toBe("first arg");

// 첫 호출의 두번째 인자는 'second arg' 입니다.
expect(someMockFunction.mock.calls[0][1]).toBe("second arg");

// 첫 호출의 반환값은 'return value' 입니다.
expect(someMockFunction.mock.results[0].value).toBe("return value");

// 함수는 두번 호출됐습니다.
expect(someMockFunction.mock.instances.length).toBe(2);

// 첫 번쟤 호출에서 생성한 인스턴스는 'name'이라는 프로퍼티를 갖고 값은 'test'입니다.
expect(someMockFunction.mock.instances[0].name).toEqual("test");

// 마지막 호출의 첫번쨰 인자는 'test'입니다.
expect(someMockFunction.mock.lastCall[0]).toBe("test");
```

## Mock 반환값

Mock 함수는 테스트 도중에 테스트 값을 코드에 주입할 수 있습니다.

```javascript
const myMock = jest.fn();
console.log(myMock());
// > undefined

myMock.mockReturnValueOnce(10).mockReturnValueOnce("x").mockReturnValue(true);
console.log(myMock(), myMock(), myMock(), myMock());
// > 10, 'x', true, true
```

Mock 함수는 매우 functional continuation-passing 스타일 코드에서도 매우 효과적입니다. 이런 방식의 코드는 테스트에서 값이 사용되기 직전에 복잡하게 실제 컴포넌트의 구현을 재현하지 않고 직접 주입할 수 있습니다. 직접 테스트되지 않는 함수 내부에 논리를 구현하려는 유혹을 피해야 합니다.

```javascript
const filterTestFn = jest.fn();
filterTestFn.mockReturnValueOnce(true).mockReturnValueOnce(false);

const result = [11, 12].filter((num) => filterTestFn(num));

console.log(result);
// > [11]
console.log(filterTestFn.mock.calls[0][0]); // 11
console.log(filterTestFn.mock.calls[1][0]); // 12
```

## Mocking Modules

유저들을 APi에서 가져오는 class가 있다고 가정합니다. class는 API를 호출하기 위해서 axios를 사용하고 모든 유저를 담고있는 `data` 어트리뷰트를 반환합니다.

```javascript
import axios from "axios";

class Users {
  static all() {
    return axios.get("/users.json").then((resp) => resp.data);
  }
}

export default Users;
```

이제 이 메서드를 API를 호출하지 않고 테스트하기 위해서 우리는 `jest.mock(...)` 함수를 사용해 axios 모듈을 자동적으로 mock 합니다.

모둘을 mock하고 나면 `mockResolvedValue`를 사용해 `.get`에서 테스트에 사용할 데이터를 원하는 형태로 반환받을 수 있습니다. 사실상 `axios.get('/users.json')`에서 원하는 가짜 데이터를 받을 수 있습니다.

```javascript
import axios from "axios";
import Users from "./users";

jest.mock("axios");

test("should fetch users", () => {
  const users = [{ name: "Bob" }];
  const resp = { data: users };
  axios.get.mockResolvedValue(resp);

  return Users.all().then((data) => expect(data).toEqual(users));
});
```

## Mocking Partials

모듈의 일부분만 mock 되고, 나머지는 유지할 수 있습니다.

```javascript
export const foo = "foo";
export const bar = () => "bar";
export default () => "baz";
```

```javascript
import defaultExport, { bar, foo } from "../foo-bar-baz";

jest.mock("../foo-bar-bax", () => {
  const originalModule = jest.requireActual("../foo-bar-baz");

  return {
    __seModule: true,
    ...orginalModule,
    default: jest.fn(() => "mocked baz"),
    foo: "mocked foo",
  };
});

test("should do a partial mock", () => {
  const defaultExportResult = defaultExport();
  expect(defaultExportResult).toBe("mocked baz");
  expect(defaultExport).toHaveBeenCalled();

  expect(foo).toBe("mocked foo");
  expect(bar()).toBe("bar");
});
```

## Mock Implementations

반환값을 지정하고 mock 함수의 구현을 전부 대체하는 기능을 넘어서는 기능이 필요한 경우도 있습니다. 이것은 `jest.fn` 혹은 mock 함수의 `mockImplementationOnce` 메서드를 사용해서 해결할 수 있습니다.

```javascript
const myMockFn = jest.fn((cb) => cb(null, true));

myMockFn((err, val) => console.log(val));
// > true
```

`mockImplementation`는 다른 모듈에서 만들어진 mock 함수의 기본 구현을 정의할 필요가 있을 때 유용합니다.

```javascript
module.exports = function () {
  // ...
};
```

```javascript
jest.mock("../foo");
const foo = require("../foo");

foo.mockImplementation(() => 42);
foo();
// > 42
```

여러번의 함수 호출이 다른 결과 값을 생성하는 경우처럼 mock 함수의 복잡한 동작을 다시 생성해야 하는 경우, `mockImplementationOnce` 메서드를 사용합니다.

```javascript
const myMockFn = jest
  .fn()
  .mockImplementationOnce((cb) => cb(null, true))
  .mockImplementationOnce((cb) => cb(null, false));

myMockFn((err, val) => console.log(val));
// > true

myMockFn((err, val) => console.log(val));
// > false
```

mock 함수의 `mockImplementationOnce`로 정의된 구현이 전부 사용되면, `jest.fn`으로 정의된 기분 구현을 실행합니다.

```javascript
const myMockFn = jest
  .fn(() => "default")
  .mockImplementationOnce(() => "first call")
  .mockImplementationOnce(() => "second call");

console.log(myMockFn(), myMockFn(), myMockFn(), myMockFn());
// > 'first call', 'second call', 'default', 'default'
```

일반적인 체이닝 되어있는 메서드가 있는 경우(그래서 매번 `this`의 반환이 필요하다면), 모든 mock에는 이를 단순화한 `.mockReturnThis()` 함수가 있습니다.

```javascript
const myObj = {
  myMethod: jest.fn().mockReturnThis(),
};

// is the same as

const otherObj = {
  myMethod: jest.fn(function () {
    return this;
  }),
};
```

## Mock Name

선탟적으로 error가 출력될 때 `jest.fn()` 대신에 출력된 함수 이름을 제공할 수 있습니다. 테스트 결과의 error 보고에서 mock 함수를 빠르게 특정하고 싶을 때 사용할 수 있습니다.

```javascript
const myMockFn = jest
  .fn()
  .mockReturnValue("default")
  .mockImplementation((scalar) => 42 + scalar)
  .mockName("add42");
```

## Custom Matchers

마지막으로, mock 함수가 어떻게 호출됐는지 테스트하데 요구사항을 줄이기 위해서 몇가지 커스텀 matcher를 추가했습니다.

```javascript
// mock 함수가 적어도 한번은 호출됐다.
expect(mockFunc).toHaveBeenCalled();

// mock 함수가 특정한 인자로 한번 이상 호출됐다.
expect(mockFunc).toHaveBeenCalledWith(arg1, arg2);

// mock 함수의 마지막 호출 인자가 특정한 인자이다.
expect(mockFunc).toHaveBeenLastCalledWith(arg1, arg2);

// mock 함수의 호출과 이름을 snapshot으로 기록한다.
expect(mockFunc).toMathcSnapshot();
```

이 matcher들은 일반적은 `.mock` 검사 프로퍼티의 문법적 설탕이다.

```javascript
// mock 함수가 적어도 한번은 호출됐다.
expect(mockFunc.mock.calls.length).toBeGreaterThan(0);

// mock 함수가 특정한 인자로 한번 이상 호출됐다.
expect(mockFunc.mock.calls).toContainEqual([arg1, arg2]);

// mock 함수의 마지막 호출 인자가 특정한 인자이다.
expect(mockFunc.mock.calls[mockFunc.mock.calls.length - 1]).toEqual([arg1, arg2]);

// mock 함수의 마지막 호출의 첫번째 인자는 42이다.
expect(mockFunc.mock.calls[mockFunc.mock.calls.length - 1][0]).toBe(42);

// snapshot은 mcok이 같은 횟수만큼 호출됐는지 체크합니다.
expect(mockFunc.mock.calls).toEqual([[arg1, arg2]]);
expect(mockFunc.getMockName()).toBe("a mock name");
```

모든 matcher 목록을 확인하고 싶다면 [레퍼런스 문서](https://jestjs.io/docs/mock-functions)를 확인하세요.
