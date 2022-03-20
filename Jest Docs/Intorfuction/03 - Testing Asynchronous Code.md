# Testing Asynchronous Code

자바스크립트에서는 코드가 비동기적으로 실행되는 것이 일반적입니다. Jest는 다른 코드로 넘어가기 전에, 비동기 코드의 완료 시점을 알아야 합니다. Jest는 몇가지 방버으로 이것을 다룹니다.

## Callback

가장 일반적인 비동기 패턴은 callback 입니다.

데이터 fetch가 완료되면 `callback(data)`를 호출하는 `fetchData(callback)` 함수를 예로 들겠습니다. 우리는 반환되는 데이터가 `peanut butter` 문자열인지 테스트해야 합니다.

```javascript
// 이렇게 하지 마세요.
test("the data id peanut butter", () => {
  function callback(data) {
    expect(data).toBe("peanut butter");
  }

  fetchData(callback);
});
```

문제는 아직 callback 함수가 실행되지 않았는데, `fetchData` 실행이 완료되지마자 테스트가 종료된다는 점입니다.

이것을 해결하는 다른 형태의 테스트가 존재합니다. 테스트에 인자가 없는 함수를 사용하는 대신에, done이라는 단일 인자를 받는 함수를 사용하세요. Jest는 done 함수가 실행되기 전까지 테스트를 종료하지 않습니다.

```javascript
test("the data is peanut butter", (done) => {
  function callback(data) {
    try {
      expect(data).toBe("peanut butter");
      done();
    } catch (error) {
      done(error);
    }
  }

  fetchData(callback);
});
```

만약 `expect`문이 실패하면 에러가 발생하고 `done()` 이 실행되지 않습니다. 만약 `done`이 영원히 불리지 않는다면 테스트는 타임아웃 에러로 실패합니다. 만약 왜 실패했는지 로그가 필요하다면, `expect`를 `try` 블록으로 감싸고 `catch` 블록을 통해 `done`으로 error를 넘길 수 있습니다.

## Promise

만약 promise를 사용한다면 좀 더 직관적인 비동기 테스트 방법이 존재합니다. 테스트에서 promise를 반환하면, Jest는 promise가 resolve 되기를 기다립니다. 만약 promise가 reject되면, 테스트는 자동적으로 실패합니다.

아까 `fetchData`를 예로 들자면, callback을 사용하는 대신에, `peanut butter`를 resolve 할 것이라고 예상되는 promise를 반한합니다.

```javascript
test("the data is peanut butter", () => {
  return fetchData().then((data) => {
    expect(data).toBe("peanut butter");
  });
});
```

promise를 반드시 반환하세요. 만약 `return` 문이 생략되면 `fetchData`에서 반환된 promise가 resolve해서 `then()`에서 callback을 실행하기 전에 테스트가 종료됩니다.

만약 promise가 reject 될 것을 예상한다면, `.catch` 메서드를 사용합니다. 특정 수의 assertion이 호출되는지 확인하려면 `expect.assertions`을 반드시 추가합니다. 그렇지 않으면, funfill 상태의 promise는 테스트를 실패로 만들지 않습니다. 즉 error가 발생하는 것을 예상했지만 error가 발생하지 않아도 테스트가 통과됩니다.

```javascript
test("the fetch fails with an error", () => {
  expect.assertions(1);
  return fetchData().catch((e) => expect(e).toMatch("error"));
});
```

## resolve / rejects

expect 문의 `.resolve` matcher를 사용하면 Jest는 promise가 resolve하기를 기다립니다. 만약 promise가 reject되면 테스트는 자동으로 실패합니다.

```javascript
test("the data is peanut butter", () => {
  return expect(fetchData()).resolves.toBe("peanut butter");
});
```

promise를 반드시 반환하세요. 만약 `return` 문이 생략되면 `fetchData`에서 반환된 promise가 resolve해서 `then()`에서 callback을 실행하기 전에 테스트가 종료됩니다.

만약 promise가 reject 될 것을 예상한다면, `.rejects` matcher를 사용합니다. 이것은 `.resolve`와 유사하게 동작합니다. 만약 promise가 fulfill이 되면 테스트는 자동으로 실패합니다.

```javascript
test("the fetch fails with an error", () => {
  return expect(fetchData()).rejects.toMatch("error");
});
```

## Async / Await

`async`와 `await`를 테스트에 사용해도 됩니다. test에 사용되는 함수 앞에 `async` 키워드를 사용하여 비동기 테스트를 진행합니다. `fetchData` 시나리오를 동일하게 예로 들자면:

```javascript
test("the data is peanut butter", async () => {
  const data = await fetchData();
  expect(data).toBe("peanut butter");
});

test("the fetch fails with an error", async () => {
  expect.assertion(1);
  try {
    await fetchData(1);
  } catch (e) {
    expect(e).toMatch("error");
  }
});
```

`async`와 `await`를 `.resolve` 혹은 `.rejects`와 함께 사용할 수 있습니다.

```javascript
test("the data is peanut butter", async () => {
  await expect(fetchData()).resolve.toBe("peanut butter");
});

test("the fetch fails with an error", async () => {
  await expect(fetchData()).rejectsa.toMatch("error");
});
```

이 경우에 `async`와 `await`는 promise 예제에서 사용한 로직과 동일한 효과적인 문법 설탕입니다.

이 모든 형식들 중에 특별히 뛰어난 것은 없으며 코드 베이스 혹은 파일을 넘나들며 혼합하고 일치시킬 수 있습니다. 테스트에 사용될 때 어떤 스타일에 더 간단한 느낌을 주는지가 가장 중요합니다.
