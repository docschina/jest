---
id: setup-teardown
title: Setup and Teardown
---

通常，在编写测试程序时，您需要在测试程序运行之前进行一些设置工作，在测试程序运行后需要进行一些整理工作。 Jest提供了辅助功能来处理此问题。
## 为多次测试重复设置

如果您有一些工作需要对多个测试重复进行，则可以使用“ beforeEach”和“ afterEach”。

例如，我们考虑一些与城市信息数据库进行交互的测试。 你必须在每个测试之前调用方法 `initializeCityDatabase()` ，同时必须在每个测试后，调用方法 `clearCityDatabase()`。 你可以这样做：

```js
beforeEach(() => {
  initializeCityDatabase();
});

afterEach(() => {
  clearCityDatabase();
});

test('city database has Vienna', () => {
  expect(isCity('Vienna')).toBeTruthy();
});

test('city database has San Juan', () => {
  expect(isCity('San Juan')).toBeTruthy();
});
```

`beforeEach` 和 `afterEach` 能够通过与 [异步代码测试](TestingAsyncCode.md) 相同的方式来处理异步代码  — 它们可以接收一个 done 参数或返回一个 promise。 例如，如果 `initializeCityDatabase()` 返回数据库初始化成功时的 promise ，我们会想到这样去返回这一 promise︰
```js
beforeEach(() => {
  return initializeCityDatabase();
});
```

## 一次性设置

在某些情况下，你只需要在文件的开头做一次设置。 当设置是异步的时，这会特别麻烦，因此您不能内联执行。 Jest 提供了 beforeAll 和 afterAll 处理这种情况

例如，如果 `initializeCityDatabase` 和 `clearCityDatabase` 都返回了 promise ，城市数据库可以在多个测试中重用，我们可以把我们的测试代码改成这样:

```js
beforeAll(() => {
  return initializeCityDatabase();
});

afterAll(() => {
  return clearCityDatabase();
});

test('city database has Vienna', () => {
  expect(isCity('Vienna')).toBeTruthy();
});

test('city database has San Juan', () => {
  expect(isCity('San Juan')).toBeTruthy();
});
```

## 作用域

默认情况下，`before` 和 `after` 块可以应用到文件中的每个测试。 此外可以通过 `describe` 块来将多个测试组合在一起。 当 `before` 和 `after` 块在 `describe` 块内部时，则其只适用于该 `describe` 块内的测试。

例如，我们不仅有一个城市数据库，而且还有一个食品数据库。 我们可以为不同的测试进行不同的设置：

```js
// 应用于此文件中的所有测试
beforeEach(() => {
  return initializeCityDatabase();
});

test('city database has Vienna', () => {
  expect(isCity('Vienna')).toBeTruthy();
});

test('city database has San Juan', () => {
  expect(isCity('San Juan')).toBeTruthy();
});

describe('matching cities to foods', () => {
  // 仅用于 describle 块级内的测试
  beforeEach(() => {
    return initializeFoodDatabase();
  });

  test('Vienna <3 sausage', () => {
    expect(isValidCityFoodPair('Vienna', 'Wiener Schnitzel')).toBe(true);
  });

  test('San Juan <3 plantains', () => {
    expect(isValidCityFoodPair('San Juan', 'Mofongo')).toBe(true);
  });
});
```

注意，顶层的 `beforeEach` 在 `describe` 块级内的 `beforeEach` 之前被执行。 这可能有助于说明所有钩子的执行顺序。

```js
beforeAll(() => console.log('1 - beforeAll'));
afterAll(() => console.log('1 - afterAll'));
beforeEach(() => console.log('1 - beforeEach'));
afterEach(() => console.log('1 - afterEach'));
test('', () => console.log('1 - test'));
describe('Scoped / Nested block', () => {
  beforeAll(() => console.log('2 - beforeAll'));
  afterAll(() => console.log('2 - afterAll'));
  beforeEach(() => console.log('2 - beforeEach'));
  afterEach(() => console.log('2 - afterEach'));
  test('', () => console.log('2 - test'));
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

## describe 和 test 块的执行顺序

Jest 会在所有真正的测试开始之前先执行测试文件里所有的 describe 处理程序（handlers）。 这是在 `before*` 和 `after*` 处理程序中而不是在describe块中执行setup和teardown的另一个原因。 一旦describe块完成，默认情况下Jest会按照在收集阶段遇到的顺序运行所有测试，等待每个测试完成并整理好，然后再继续。

考虑以下的示例性测试文件和输出:

```js
describe('outer', () => {
  console.log('describe outer-a');

  describe('describe inner 1', () => {
    console.log('describe inner 1');
    test('test 1', () => {
      console.log('test for describe inner 1');
      expect(true).toEqual(true);
    });
  });

  console.log('describe outer-b');

  test('test 1', () => {
    console.log('test for describe outer');
    expect(true).toEqual(true);
  });

  describe('describe inner 2', () => {
    console.log('describe inner 2');
    test('test for describe inner 2', () => {
      console.log('test for describe inner 2');
      expect(false).toEqual(false);
    });
  });

  console.log('describe outer-c');
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

## 通用建议

如果一个测试失败了，第一件要检查的事就是，当只有这一条测试在运行时，它是否是失败的。要使用 Jest 只运行一个测试，可暂时将 `test` 命令更改为 `test.only`

```js
test.only('this will be the only test that runs', () => {
  expect(true).toBe(false);
});

test('this test will not run', () => {
  expect('A').toBe('A');
});
```

如果您有一个测试，当它作为一个更大的套件的一部分运行时经常失败，但是当您单独运行它时却没有失败，那么很有可能来自不同测试的某些东西正在干扰这个测试。通常可以通过使用 `beforeach` 清除某些共享状态来解决此问题。如果不确定是否修改了某些共享状态，也可以尝试使用 `beforeach` 记录数据。
