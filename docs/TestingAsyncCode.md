---
id: asynchronous
title: 测试异步代码
---

异步代码在Javascript中是很常见的。当在运行异步代码时，Jest有多种方法可以知道代码什么时候可以完成测试，进而再去进行另一项测试。

##  回调函数

回调函数是最常见的异步处理方式.

举个例子, 有一个`fetchData(callback)`函数用来请求某些数据，当请求完成时需要调用`callback(data)`回调函数。你想测试返回的数据是否是`'peanut butter'`这个字符串。

默认情况下，Jest测试会在执行结束后完成，这意味着测试没有按照预期工作。

```js
// 不要这样做！
test('the data is peanut butter', () => {
  function callback(data) {
    expect(data).toBe('peanut butter');
  }

  fetchData(callback);
});
```

问题在于测试会在`fetchData`请求结束后，回调函数未被调用之前完成。

为了解决这个问题，采取了另外一种`test`的方式，通过使用单一函数参数`done`，Jest会等到`done`被调用之后结束测试，而不是把测试放在一个空参的函数中。

```js
test('the data is peanut butter', done => {z c
  function callback(data) {
    try {
      expect(data).toBe('peanut butter');
      done();
    } catch (error) {
      done(error);
    }
  }

  fetchData(callback);
});
```

如果`done()`没有被调用（或者出现超时错误），本次测试将会失败，这是我们期望的结果。

如果`expect`执行失败，`done()`不会被调用，并且会抛出一个错误。如果我们想在测试记录中了解为什么出错，需要将`expect`放入`try`中，然后将错误传递给`catch` 中的`done`函数。否则，最后会显示一个模糊的超时错误，而不是`expect(data)`中接收到的数据。

## Promises

如果你在代码中使用了promise，那这里有一种更直接的方式去处理异步测试。通过在测试中返回一个promise，Jest会等待promise的resolve被执行，如果promise状态变为rejected，本次测试就会自动失败。

举个例子，`fetchData` 不使用回调函数，而是返回一个promise，resolve的值为一个字符串`'peanut butter'`，我们可以这样进行测试：

```js
test('the data is peanut butter', () => {
  return fetchData().then(data => {
    expect(data).toBe('peanut butter');
  });
});
```

一定不要忘了返回promise，如果忘了return语句，测试会在`fetchData`中的promise被resolve以及then函数执行之前结束。

如果你预期一个promise的状态会变为rejected，可以使用`.catch`方法处理。确保添加`expect.assertions`去验证一定数量的断言被调用，否则，一个fullfilled状态的promise不会导致测试失败。

```js
test('the fetch fails with an error', () => {
  expect.assertions(1);
  return fetchData().catch(e => expect(e).toMatch('error'));
});
```

## `.resolves` / `.rejects`

你也可以使用`.resolves`在你的expect语句中，Jest一直等待直到promise执行resolve。如果promise的状态变为rejected，那么本次测试就会自动失败。

```js
test('the data is peanut butter', () => {
  return expect(fetchData()).resolves.toBe('peanut butter');
});
```

一定不要忘了返回断言，如果忘了return语句，测试会在`fetchData`中的promise被resolve以及then函数执行之前结束。

如果你预期一个promise的状态会变为rejected，可以使用`.rejects`处理. 它的使用方式与`.resolves`相似。如果promise状态变为fullfilled，那么本次测试就会自动失败。

```js
test('the fetch fails with an error', () => {
  return expect(fetchData()).rejects.toMatch('error');
});
```

## Async/Await

或者，你也可以在测试中使用`async` 和 `await`。写一个async测试，需要将`async` 关键词放在函数前面传递给`test`，举个例子，相同的`fetchData`方案可以这样进行测试：

```js
test('the data is peanut butter', async () => {
  const data = await fetchData();
  expect(data).toBe('peanut butter');
});

test('the fetch fails with an error', async () => {
  expect.assertions(1);
  try {
    await fetchData();
  } catch (e) {
    expect(e).toMatch('error');
  }
});
```

你可以将 `async` 和 `await` 与 `.resolves` 或者 `.rejects`结合在一起.

```js
test('the data is peanut butter', async () => {
  await expect(fetchData()).resolves.toBe('peanut butter');
});

test('the fetch fails with an error', async () => {
  await expect(fetchData()).rejects.toThrow('error');
});
```

在这些例子中，`async` 和 `await`是promise的语法糖。

这些方案中没有一个相较于其他方案更加优秀，你可以通过在代码库甚至单个文件中使用和比较，这取决于哪一种方式让你觉得测试更加简单。
