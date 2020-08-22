---
id: timer-mocks
title: 模拟计时器
---

原生的计时器函数（如 `setTimeout`, `setInterval`, `clearTimeout`, `clearInterval`）在测试环境下并不理想，因为它们需要依靠真实时间流逝。Jest 可以通过函数来把计时器改变成能够让你控制时间流逝的计时器。[太棒了！](https://www.youtube.com/watch?v=QZoJ2Pt27BY)

```javascript
// timerGame.js
'use strict';

function timerGame(callback) {
  console.log('Ready....go!');
  setTimeout(() => {
    console.log("Time's up -- stop!");
    callback && callback();
  }, 1000);
}

module.exports = timerGame;
```

```javascript
// __tests__/timerGame-test.js
'use strict';

jest.useFakeTimers();

test('waits 1 second before ending the game', () => {
  const timerGame = require('../timerGame');
  timerGame();

  expect(setTimeout).toHaveBeenCalledTimes(1);
  expect(setTimeout).toHaveBeenLastCalledWith(expect.any(Function), 1000);
});
```

在上述代码中，我们通过调用 `jest.useFakeTimers();` 开启了假计时器，这样一来 setTimeout 和其它计时器的函数就都成为模拟的了。如果在同一个文件，或者同一个描述块(Describe Block)中执行多段测试代码，`jest.useFakeTimers();` 可以在每一段测试代码前手动调用，或者使用如 `beforeEach` 这样的设置函数。如果不这么做，就会导致内部的引用计数器不会重置。

## 执行所有的计时器

在本模块内容中，我们想要写的另一个测试是在一秒钟后断言一个回调函数被执行。我们将会使用到 Jest 的计时器控制 API，在测试过程中加速时间进程，以此实现测试内容。

```javascript
test('calls the callback after 1 second', () => {
  const timerGame = require('../timerGame');
  const callback = jest.fn();

  timerGame(callback);

  // 在此时间点上，回调函数未被调用
  expect(callback).not.toBeCalled();

  // 加速时间直到所有计时器都被执行
  jest.runAllTimers();

  // 现在回调函数应该已经被调用了！
  expect(callback).toBeCalled();
  expect(callback).toHaveBeenCalledTimes(1);
});
```

## 执行挂起的计时器(Pending Timer)

也存在着你有一个递归计时器的场景——递归计时器是指一个计时器在它自己的回调函数中设置了一个新的计时器。对于这一类来说，执行所有计时器会导致死循环。因此类似 `jest.runAllTimers()` 的代码就不是我们想要的。对于这种情况，你可能会使用到 `jest.runOnlyPendingTimers()`：

```javascript
// infiniteTimerGame.js
'use strict';

function infiniteTimerGame(callback) {
  console.log('Ready....go!');

  setTimeout(() => {
    console.log("Time's up! 10 seconds before the next game starts...");
    callback && callback();

    // 下一个 game 在 10 秒后开始
    setTimeout(() => {
      infiniteTimerGame(callback);
    }, 10000);
  }, 1000);
}

module.exports = infiniteTimerGame;
```

```javascript
// __tests__/infiniteTimerGame-test.js
'use strict';

jest.useFakeTimers();

describe('infiniteTimerGame', () => {
  test('schedules a 10-second timer after 1 second', () => {
    const infiniteTimerGame = require('../infiniteTimerGame');
    const callback = jest.fn();

    infiniteTimerGame(callback);

    // 在此时间点上，应该会单独调用一次 setTimeout
    // 来设置 game 在 1 秒钟后结束
    expect(setTimeout).toHaveBeenCalledTimes(1);
    expect(setTimeout).toHaveBeenLastCalledWith(expect.any(Function), 1000);

    // 加速时间，并且只结束当前挂起的计时器
    // (而不是在此过程中新创建出的任何计时器)
    jest.runOnlyPendingTimers();

    // 在此时间点，我们 1 秒的计时器应该已经调用了回调函数
    expect(callback).toBeCalled();

    // 并且它应该已经创建了一个新的计时器
    // 设置 10 秒钟后执行 game over
    expect(setTimeout).toHaveBeenCalledTimes(2);
    expect(setTimeout).toHaveBeenLastCalledWith(expect.any(Function), 10000);
  });
});
```

## 根据时间推进计时器

##### `runTimersToTime` 已经在 Jest **22.0.0** 中更名为 `advanceTimersByTime`

另一种可能是使用 `jest.advanceTimersByTime(msToRun)`。当这个 API 被调用时，所有的计时器都会向前推进 `msToRun` 毫秒。所有挂起的“宏任务”，即通过 setTimeout() 和 setInterval() 加入到队列中的，并在此时间片上将会执行的任务，会被执行。此外，如果这些宏任务又设置了同样会在这个时间片中执行的新宏任务，这些新的宏任务会直到 msToRun 毫秒内没有其它剩余宏任务在队列中时执行。

```javascript
// timerGame.js
'use strict';

function timerGame(callback) {
  console.log('Ready....go!');
  setTimeout(() => {
    console.log("Time's up -- stop!");
    callback && callback();
  }, 1000);
}

module.exports = timerGame;
```

```javascript
it('calls the callback after 1 second via advanceTimersByTime', () => {
  const timerGame = require('../timerGame');
  const callback = jest.fn();

  timerGame(callback);

  // 在此时间点上，回调函数未被调用
  expect(callback).not.toBeCalled();

  // 加速时间，直到所有计时器都被执行
  jest.advanceTimersByTime(1000);

  // 现在回调函数应该已经被调用了！
  expect(callback).toBeCalled();
  expect(callback).toHaveBeenCalledTimes(1);
});
```

最后，偶尔在某些测试代码中，能够清除所有挂起的计时器会很有用。这一点，我们有 `jest.clearAllTimers()`。

此例中的代码可以在 [example/timer](https://github.com/facebook/jest/tree/master/examples/timer) 中查看。
