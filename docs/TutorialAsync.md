
# An Async Example


首先, 像 [Getting Started](GettingStarted.md#using-babel) 里面所说的那样, 启用babel的支持。

让我们实现一个从 API 获取用户数据并返回用户名的模块。

```js
// user.js
import request from './request';

export function getUserName(userID) {
  return request('/users/' + userID).then(user => user.name);
}
```

在上述实现中，我们希望 `request.js` 模块返回一个 promise。 通过链式调用 `then` 来接收用户名。

现在思考一下如何实现一个 `request.js` 模块，使其可以从网络中获取用户数据：

```js
// request.js
const http = require('http');

export default function request(url) {
  return new Promise(resolve => {
    // 这是一个 HTTP 请求的例子, 用来从 API 中获取用户信息。
    // 这个模块正在 __mocks__/request.js 中进行模拟
    http.get({path: url}, response => {
      let data = '';
      response.on('data', _data => (data += _data));
      response.on('end', () => resolve(data));
    });
  });
}
```

因为我们不想在测试中访问网络，所以我们在 `__mocks__` 文件夹下 （这个文件夹名是大小写敏感的，写成 `__MOCKS__` 将不起作用） 创建一个 `request.js` 模块去手动模拟网络请求。它可能看起来像这样：

```js
// __mocks__/request.js
const users = {
  4: {name: 'Mark'},
  5: {name: 'Paul'},
};

export default function request(url) {
  return new Promise((resolve, reject) => {
    const userID = parseInt(url.substr('/users/'.length), 10);
    process.nextTick(() =>
      users[userID]
        ? resolve(users[userID])
        : reject({
            error: 'User with ' + userID + ' not found.',
          }),
    );
  });
}
```

现在为我们的异步功能编写一个测试。

```js
// __tests__/user-test.js
jest.mock('../request');

import * as user from '../user';

// promise 的 assertion 必须被返回。
it('works with promises', () => {
  expect.assertions(1);
  return user.getUserName(4).then(data => expect(data).toEqual('Mark'));
});
```

我们调用 `jest.mock('../request')` 来告诉 Jest 使用我们手动创建的模拟数据 `it` 希望返回值是一个 resolved 状态的 Promise。 你可以链接任意数量的 Promise, 也可以在任何时刻调用 `expect`, 只要在最后返回一个 Promise。

## `.resolves`

有一种不那么冗长的方式是使用 `resolves`，让它与任何其他匹配器一起展开 一个 fulfilled 状态的 promise 的值。如果 promise 状态是 rejected，则 assertion 将失败。

```js
it('works with resolves', () => {
  expect.assertions(1);
  return expect(user.getUserName(5)).resolves.toEqual('Paul');
});
```

## `async`/`await`

使用 `async`/`await` 语法编写测试很容易。可以写一个和上面实例一致的测试:

```js
// async/await 可以被使用
it('works with async/await', async () => {
  expect.assertions(1);
  const data = await user.getUserName(4);
  expect(data).toEqual('Mark');
});

// async/await 也能和 resolves 一起使用.
it('works with async/await and resolves', async () => {
  expect.assertions(1);
  await expect(user.getUserName(5)).resolves.toEqual('Paul');
});
```

为了使得测试中支持 `async`/`await`, 需要安装 [`@babel/preset-env`](https://babeljs.io/docs/en/babel-preset-env) 并且需要在 `babel.config.js` 文件中启用这个属性。

## 错误处理

可以使用 `.catch` 方法处理错误。确保添加了 `expect.assertions` 来验证是否调用了一定数量的 assertions 。否则，一个 fulfilled 状态的 promise 不会使测试失败：

```js
// 使用 Promise.catch 测试异步错误
it('tests error with promises', () => {
  expect.assertions(1);
  return user.getUserName(2).catch(e =>
    expect(e).toEqual({
      error: 'User with 2 not found.',
    }),
  );
});

// 或者使用 async/await.
it('tests error with async/await', async () => {
  expect.assertions(1);
  try {
    await user.getUserName(1);
  } catch (e) {
    expect(e).toEqual({
      error: 'User with 1 not found.',
    });
  }
});
```

## `.rejects`

`.rejects` 和 `.resolves` 一样，都能帮助程序。如果 promise 状态是 fulfilled，测试将自动失败。
`expect.assertions(number)` 不是必需的，但建议用于验证在测试期间是否调用了一定数量的 [assertions](https://jestjs.io/docs/en/expect#expectassertionsnumber) 。 否则很容易忘记 `return` 或者 `await` `.resolves` 的 assertion.

```js
// 使用 `.rejects` 测试异步错误
it('tests error with rejects', () => {
  expect.assertions(1);
  return expect(user.getUserName(3)).rejects.toEqual({
    error: 'User with 3 not found.',
  });
});

// 或者使用 async/await 和 `.rejects`.
it('tests error with async/await and rejects', async () => {
  expect.assertions(1);
  await expect(user.getUserName(3)).rejects.toEqual({
    error: 'User with 3 not found.',
  });
});
```

这个例子的源代码可以在 [examples/async](https://github.com/facebook/jest/tree/master/examples/async) 中找到。

如果你想要测试定时器，如 `setTimeout`,可以看看 [Timer mocks](TimerMocks.md) 的文档。
