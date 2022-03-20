# Using Matchers

Jest는 "matchers"를 사용해서 여러가지 방법으로 값을 테스트합니다. 이 문서는 일반적으로 사용되는 matcher들을 소개합니다. 모든 내요을 보고싶다면 [expect API 문서](https://jestjs.io/docs/expect)를 확인하세요.

## 일반적인 Matchers

값이 정확히 같은지 확인하는 것은 값을 검사하는데 가장 간단한 방법입니다.

```javascript
test("two plus two is four", () => {
  expect(2 + 2).toBe(4);
});
```

코드에서 `expect(2 + 2)`는 "expectation" 객체를 반환합니다. 일반적으로 expectation 객체로 matchers를 호출하는 것 외에는 할 것이 많지 않습니다. 코드에서 `.toBe(4)`는 matcher입니다. Jest가 실행될 때, 실패한 모든 matcher들을 추적하고 에러 메세지를 보여줍니다.

```javascript
test("object assignment", () => {
  const data = { one: 1 };
  data["two"] = 2;
  expect(data).toEqual({ one: 1, two: 2 });
});
```

`toEqual`은 재귀적으로 객체와 배열의 모든 필드를 체크합니다.

반대의 경우도 matcher로 테스트할 수 있습니다:

```javascript
test("adding positive numbers is not zero", () => {
  for (let a = 1; a < 10; a++) {
    for (let b = 1; b < 10; b++) {
      expect(a + b).not.toBe(0);
    }
  }
});
```

## 진실성

때로는 `undefined`, `null` 그리고 `false`를 구분해야할 필요가 있습니다. 하지만 구분해야할 필요가 없을 때도 있습니다.

- `toBeNull`은 오로지 `null`과 매치합니다.
- `toBeUndefuned`는 오로지 `undefined`와 매치합니다.
- `toBeDefined`는 `toBeUndefined`와 반대입니다.
- `toBeTruthy`는 `if` 조건문이 참으로 판정하는 모든 것과 매치합니다.
- `toBeFalsy`는 `if` 조건문이 거짓으로 판정하는 모든 것과 매치합니다.

```javascript
test("null", () => {
  const n = null;
  expect(n).toBeNull();
  expect(n).toBeDefined();
  expect(n).not.toBeUndefined();
  expect(n).not.toBeTruthy();
  expect(n).toBeFalsy();
});

test("zero", () => {
  const z = 0;
  expect(z).not.toBeNull();
  expect(z).toBeDefined();
  expect(z).not.toBeUndefined();
  expect(z).not.toBeTruthy();
  expect(z).toBeFalsy();
});
```

코드에서 수행하고자하는 작업과 가장 적함한 matcher를 사용해야 합니다.

## 숫자

숫자를 비교하는 대부분의 방법은 동일한 matcher를 가지고 있습니다.

```javascript
test("two plus two", () => {
  const value = 2 + 2;
  expect(value).toBeGreaterThan(3);
  expect(value).toBeGreaterThanOrEqual(3.5);
  expect(value).toBeLessThan(5);
  expect(value).toBeLessThanOrEqual(4.5);

  expect(value).toBe(4);
  expect(value).toEqaul(4);
});
```

부동 소수점 비교에서는 `toEqual` 대신에 `toBeCloseTo` 사용합니다.

```javascript
test("adding floating point numbers", () => {
  const value = 0.1 + 0.2;
  expect(value).toBeCloseTo(0.3);
});
```

## 문자열

`toMathc`를 사용해서 정규표현식으로 문자열을 체크할 수 있습니다:

```javascript
test("there is np I in team", () => {
  expect("team").not.toMatch(/I/);
});

test('but there is a "stop" in Christoph', () => {
  expect("Christoph").toMatch(/stop/);
});
```

## 배열과 이터러블

`toContain`을 사용해서 배열 또는 이터러블에 특정 아이템이 포함됐는지 체크할 수 있습니다.

```javascript
const shoppingList = ["diapers", "kleenex", "trash boags", "paper towels", "milk"];

test("the shopping list has milk on it", () => {
  expect(shoppingList).toContain("milk");
  expect(new Set(shoppingList)).toContain("milk");
});
```

## 예외

만약 특정 함수가 실행될 때 에러가 발생하는 것을 테스트하고 싷다면, `toThrow`를 사용하세요.

```javascript
function compileAndroidCode() {
  throw new Error("you are using the wrong JDK");
}

test("compioling android goes as expected", () => {
  expect(() => compileAndroidCode()).toThrow();
  expect(() => compileAndroidCode()).toTrhow(Error);

  // 에러 메시지를 체크할 수도 있습니다.
  expect(() => compileAndroidCode()).toTrhow("you are using the wrong JDK");
  expect(() => compileAndroidCode()).toThrow(/JDK/);
});
```

> 참고: 에러가 발생하는 함수는 래핑 함수 내부에서 호출해야 합니다. 그렇지 않으면 `toThrow`는 실패합니다.

## 그리고 나머지

지금까지는 맛보기입니다. matchers 목록을 완성하고 싶으면 [레퍼런스 문서](https://jestjs.io/docs/expect)를 확인하세요.
