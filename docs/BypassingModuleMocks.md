---
id: bypassing-module-mocks
title: 绕过模块模拟
---

Jest允许你在测试中模拟出整个模块，如果你的代码正确地从该模块调用函数，这对于测试非常有用。但是，有时你可能希望在 _测试文件_ 中使用部分模拟模块，在这种情况下，你希望访问代码的原始实现，而非其模拟版本。

考虑为如下的 `createUser` 函数编写测试用例：

```javascript
// createUser.js
import fetch from 'node-fetch';

export const createUser = async () => {
  const response = await fetch('http://website.com/users', {method: 'POST'});
  const userId = await response.text();
  return userId;
};
```

在你所编写的测试中，你将会模拟 `fetch` 函数，这样即可确保它在没有实际发出网络请求的情况下被调用。但是，你还需要模拟`fetch`的返回值，其包含`Response`（包装在`Promise`中），因为后面的函数使用它来获取所创建的用户的ID。因此，你可以先尝试编写这样的测试：

```javascript
jest.mock('node-fetch');

import fetch, {Response} from 'node-fetch';

import {createUser} from './createUser';

test('createUser calls fetch with the right args and returns the user id', async () => {
  fetch.mockReturnValue(Promise.resolve(new Response('4')));

  const userId = await createUser();

  expect(fetch).toHaveBeenCalledTimes(1);
  expect(fetch).toHaveBeenCalledWith('http://website.com/users', {
    method: 'POST',
  });
  expect(userId).toBe('4');
});
```

However, if you ran that test you would find that the `createUser` function would fail, throwing the error: `TypeError: response.text is not a function`. This is because the `Response` class you've imported from `node-fetch` has been mocked (due to the `jest.mock` call at the top of the test file) so it no longer behaves the way it should.
然而，如果运行如上测试，你会发现`createUser`函数将失败，并抛出错误信息`TypeError: response.text is not a function`。这是因为从`node-fetch`导入的`Response`类已被模拟（由于`jest.mock`在测试文件的顶部调用），这样它就不再按照它规定的方式运行了。

为了解决这样的问题，Jest提供了一个`jest.requireActual`辅助工具。要使上述测试正常工作，需对测试文件中的导入进行以下更改：

```javascript
// BEFORE
jest.mock('node-fetch');
import fetch, {Response} from 'node-fetch';
```

```javascript
// AFTER
jest.mock('node-fetch');
import fetch from 'node-fetch';
const {Response} = jest.requireActual('node-fetch');
```

这允许测试文件从 `node-fetch` 导入实际的 `Response` 对象，而不是模拟版本。这意味着测试将正确通过。
