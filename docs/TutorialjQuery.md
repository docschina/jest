---
id: tutorial-jquery
title: DOM 操作
---

通常认为难以测试的另一类功能是直接操作 DOM 的代码。下面是一段 jQuery 代码，监听一个 click 事件，在回调函数中异步获取一些数据并设置 span 元素中的内容，让我们看看如何测试下面这段代码。

```javascript
// displayUser.js
'use strict';

const $ = require('jquery');
const fetchCurrentUser = require('./fetchCurrentUser.js');

$('#button').click(() => {
  fetchCurrentUser(user => {
    const loggedText = 'Logged ' + (user.loggedIn ? 'In' : 'Out');
    $('#username').text(user.fullName + ' - ' + loggedText);
  });
});
```

同样，我们在 `__tests__/` 文件夹中创建一个测试文件：

```javascript
// __tests__/displayUser-test.js
'use strict';

jest.mock('../fetchCurrentUser');

test('displays a user after a click', () => {
  // 设置 document 中 body 的内容
  document.body.innerHTML =
    '<div>' +
    '  <span id="username" />' +
    '  <button id="button" />' +
    '</div>';

  // 这个模块需要引用依赖
  require('../displayUser');

  const $ = require('jquery');
  const fetchCurrentUser = require('../fetchCurrentUser');

  // 告诉 fetchCurrentUser 使用一些数据
  // 来模拟函数自动调用其回调函数
  fetchCurrentUser.mockImplementation(cb => {
    cb({
      fullName: 'Johnny Cash',
      loggedIn: true,
    });
  });

  // 使用 jquery 模拟点击按钮
  $('#button').click();

  // 验证 fetchCurrentUser 函数被调用了，并且
  // #username span 的内部文本按照预料中的进行了更新。
  expect(fetchCurrentUser).toBeCalled();
  expect($('#username').text()).toEqual('Johnny Cash - Logged In');
});
```

上述代码中被测试的函数在 `#button` 这个 DOM 元素上添加了一个事件监听器，因此我们需要为测试正确设置 DOM。Jest 附带了 `jsdom`，它可以像在浏览器中一样模拟 DOM 环境。这意味着我们在测试中调用的每个 DOM API 都可以像在浏览器中一样被观察到！

我们模拟 `fetchCurrentUser.js`，因此我们的测试不会发出真正的网络请求，而是在本地模拟数据。这样可以确保我们的测试可以在几毫秒内完成，而不需要几秒钟，并且可以保证单元测试的快速迭代。

可以在 [examples/jquery](https://github.com/facebook/jest/tree/master/examples/jquery) 上找到此示例的代码。
