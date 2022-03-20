# Setup and Teardown

테스트를 작성하는 동안 테스트가 실행되기 전에 일어나야 할 설정 작업과 테스트가 끝난 후에 일어나야 하는 마무리 작업이 있는 경우가 많습니다. Jest는 이를 도와주는 헬퍼 함수를 제공합니다.

## 여러 테스트를 위한 반복적인 설정 작업

만약 수많은 테스트를 위한 반복적인 작업이 있다면, `beforeEach`와 `afterEach`를 사용할 수 있습니다.

도시 데이터 베이스와 상호작용하는 몇가지 테스트를 예로 들겠습니다. `initializeDatabase()`는 메서드를 각 테스트 전에 반드시 실행해야하고, `clearCityDatabase()`는 각 테스트 후에 실행해야 합니다.

```javascript
beforeEach(() => {
  initializeCityDatabase();
});

afterEach(() => {
  clearCityDatabase();
});

test("city database has Vienna", () => {
  expect(isCity("Vienna")).toBeTruthy();
});

test("city database has San Juan", () => {
  expect(isCity("San Juan")).toBeTruthy();
});
```

`beforeEach`와 `afterEach`는 비동기 코드에서 [비동기 코드 테스트](https://jestjs.io/docs/asynchronous)와 동일한 방법으로 사용할 수 있습니다. `done` 파라미터를 받거나 promise를 반환할 수 있습니다. 예를 들어, `initializeCityDatabase()`가 promise를 반환하고 데이터베이스가 초기화되면 resolve 된다고 가정합니다:

```javascript
beforeEach(() => {
  return initializeCityDatabase();
});
```

## 원타임 설정

어떤 경우에는 파일의 시작지점에 단 한번의 설정만 필요할 수도 있습니다. 특히 설정이 비동기라면 귀찮아집니다. Jest는 `beforeAll`과 `afterAll`을 제공해 이 상황을 해결합니다.

만약 `initializeCityDatabase`와 `clearCityDatabase`가 promise를 반환하고, 도시 데이터베이스는 테스트간에 재사용 될 수 있다면, 코드를 다음처럼 변경할 수 있습니다:

```javascript
beforeAll(() => {
  return initializeCityDatabase();
});

afterAll(() => {
  return clearCityDatabase();
});

test("city database has Vienna", () => {
  expect(isCity("Vienna")).toBeTruthy();
});

test("city database has San Juan", () => {
  expect(isCity("San Juan")).toBeTruthy();
});
```

## 스코핑

기본적으로 `beforeAll`과 `afterAll` 블록은 파일 내부 모든 테스트에 적용됩니다. `describe` 블록을 사용해서 테스트를 그룹화할 수 있습니다.`describe` 내부의 `beforeAll`과 `afterAll`은 블럭 내부 테스트에만 적용됩니다.

```javascript
beforeEach(() => {
  return initializeCityDatabase();
});

test("city database has Vienna", () => {
  expect(isCity("Vienna")).toBeTruthy();
});

test("city database has San Juan", () => {
  expect(isCity("San Juan")).toBeTruthy();
});

describe("mathcing cities to foods", () => {
  beforeEach(() => {
    return initializeCityDatabase();
  });

  test("Vienna <3 veal", () => {
    expect(isValidCityFoodPair("Vienna", "Wiener Schnitzel")).toBe(true);
  });

  test("San Juan <3 plantains", () => {
    expect(isValidCityFoodPair("San Juan", "Mofongo")).toBe(true);
  });
});
```

상위 레벨 `beforeEach`는 `describe` 내부의 `beforeEach`보다 먼저 실행된다는 것에 주의하세요. 이것은 모든 hooks의 실행 순서를 그리는데 도움이 됩니다.

```javascript
beforeAll(() => console.log("1 - beforeAll"));
afterAll(() => console.log("1 - afterAll"));
beforeEach(() => console.log("1 - beforeEach"));
afterEach(() => console.log("1 - afterEach"));
test("", () => console.log("1 - test"));
describe("Scoped / Nested block", () => {
  beforeAll(() => console.log("2 - beforeAll"));
  afterAll(() => console.log("2 - afterAll"));
  beforeEach(() => console.log("2 - beforeEach"));
  afterEach(() => console.log("2 - afterEach"));
  test("", () => console.log("2 - test"));
});

// 1 - beforeAll
// 1 - beforeEach
// 1 - test
// 1 - afterEach
// 2 - beforeAll
// 1 - beforeEach
// 2 - beforeEach
// 2 - test
// 2 - afterEach
// 1 - afterEach
// 2 - afterAll
// 1 - afterAll
```

## describe 블록과 test 블록의 실행 순서

Jest는 테스트 파일 내부의 모든 describe 핸들러를 실제 테스트를 시작하기 전에 실행합니다. 이 현상은 `decribe` 내부가 아닌 `before*`와 `after*` 내부에서 설정과 분해를 수행해야하는 또 다른 이유입니다. describe 블록이 완료되면, 기본적으로 Jest는 수집 단계에서 발생한 순서대로 모든 테스트를 연속적으로 실행하고, 각 테스트가 완료되고 정리될 때까지 기다린 후 다음 테스트를 실행합니다.

```javascript
describe("outer", () => {
  console.log("describe outer-a");

  describe("describe inner 1", () => {
    console.log("describe inner 1");
    test("test 1", () => {
      console.log("test for describe inner 1");
      expect(true).toEqual(true);
    });
  });

  console.log("describe outer-b");

  test("test 1", () => {
    console.log("test for describe outer");
    expect(true).toEqual(true);
  });

  describe("describe inner 2", () => {
    console.log("describe inner 2");
    test("test for describe inner 2", () => {
      console.log("test for describe inner 2");
      expect(false).toEqual(false);
    });
  });

  console.log("describe outer-c");
});

// describe outer-a
// describe inner 1
// describe outer-b
// describe inner 2
// describe outer-c
// test for describe inner 1
// test for describe outer
// test for describe inner 2
```

## 일반적인 조언

만약 테스트가 실패하면 자장 먼저 확인해야할 것은 해당 테스트 하나만 단일로 실행해도 실패하는지 여부 입니다. Jest에서 단 하나의 테스트만 실행하려면, 일시적으로 `test`를 `test.only` 명령어로 변경하면 됩니다.

```javascript
test.only("this will be the only test that runs", () => {
  expect(true).toBe(false);
});

test("this test will not run", () => {
  expect("A").toBe("A");
});
```

만약 큰 제품에서 자주 실패하는 테스트가 있지만 단독으로 실행했을 때 실패하지 않는다면, 다른 테스트의 무언가가 이 테스트를 방해하고 있다고 의심해야 합니다. 종종 `beforeEach`에서 공유하는 상태를 지워서 해결되는 경우가 있습니다. 일부 공유 상태가 변경되고 있는지 확실하지 않다면, `beforeEach`에서 로그를 남겨볼 수 있습니다.
