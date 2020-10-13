---
id: troubleshooting
title: 解决疑难
---

咦？遇到麻烦了吗？使用本节指南来解决Jest的相关问题！

## 测试失败了，但你却不知为何？

尝试使用Node内置的调试功能。注意：该功能仅支持Node.js 8以上版本

在你的任一个测试中声明一个`debugger;`，然后在你的项目目录中执行如下：

```bash
node --inspect-brk node_modules/.bin/jest --runInBand [any other arguments here]
或者在Window系统：
node --inspect-brk ./node_modules/jest/bin/jest.js --runInBand [any other arguments here]
```

这将让Jest运行在一个Node进程之中，该进程能被外部的调试器所连接。需要注意的是，这一进程直到调试器连接到它为止都处于暂停状态。

若想要在谷歌浏览器（或者其他基于Chromium内核的浏览器）中调试，首先打开浏览器并转到`chrome://inspect`，然后点击“打开Node专用开发工具”，它将为你提供一个可以连接到的可用节点实例的列表。运行上述命令后，点击终端中所显示的地址（通常类似：`localhost:9229`），你将能够使用谷歌的开发者工具进行Jest调试。

接着谷歌开发者工具将展现出来，并在Jest CLI脚本的第一行设置一个断点（这样做是为了让你有时间打开开发者工具，并防止Jest在你准备好之前就执行）。然后，单击屏幕右上角类似“播放”按钮的地方继续执行。当Jest运行包含 `debugger` 语句的测试时，执行过程将暂停，此时你可以检查当前作用域并调用堆栈。

> 注意：`--runInBand` cli选项确保Jest在同一个进程中运行测试，而不是为单个测试生成进程。通常情况下，Jest 并行化测试会跨进程执行，但是很难同时调试多个进程。

## 在VS Code中调试

使用[Visual Studio Code's](https://code.visualstudio.com)中内置的[调试器](https://code.visualstudio.com/docs/nodejs/nodejs-debugging)调试est测试的方法有多种。

要连接内置调试器，请按照如下步骤运行测试：

```bash
node --inspect-brk node_modules/.bin/jest --runInBand [any other arguments here]
or on Windows
node --inspect-brk ./node_modules/jest/bin/jest.js --runInBand [any other arguments here]
```

然后使用以下`launch.json`配置来连接VS Code的调试器：

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "attach",
      "name": "Attach",
      "port": 9229
    }
  ]
}
```

要自动启动并连接到运行测试的进程，请使用以下配置：

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug Jest Tests",
      "type": "node",
      "request": "launch",
      "runtimeArgs": [
        "--inspect-brk",
        "${workspaceRoot}/node_modules/.bin/jest",
        "--runInBand"
      ],
      "console": "integratedTerminal",
      "internalConsoleOptions": "neverOpen",
      "port": 9229
    }
  ]
}
```

或者在Windows系统中使用：

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug Jest Tests",
      "type": "node",
      "request": "launch",
      "runtimeArgs": [
        "--inspect-brk",
        "${workspaceRoot}/node_modules/jest/bin/jest.js",
        "--runInBand"
      ],
      "console": "integratedTerminal",
      "internalConsoleOptions": "neverOpen",
      "port": 9229
    }
  ]
}
```

如果您使用的是Facebook的[`create-react-app`]（https://github.com/facebookincubator/create-react-app），则可以使用以下配置调试Jest测试：

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug CRA Tests",
      "type": "node",
      "request": "launch",
      "runtimeExecutable": "${workspaceRoot}/node_modules/.bin/react-scripts",
      "args": ["test", "--runInBand", "--no-cache", "--env=jsdom"],
      "cwd": "${workspaceRoot}",
      "protocol": "inspector",
      "console": "integratedTerminal",
      "internalConsoleOptions": "neverOpen"
    }
  ]
}
```

更多在Node上调试的技巧可查阅[此处](https://nodejs.org/api/debugger.html)。

## 在WebStorm中调试

在[WebStorm](https://www.jetbrains.com/webstorm/)中调试Jest测试的最简单方法就是使用`Jest run/debug configuration`命令。它将启动测试并自动连接调试器。

在WebStorm菜单 `Run`”中，选择`Edit Configurations...`，然后单击`+`并选择`Jest`，可选地指定Jest配置文件、附加选项和环境变量。保存配置，在代码中放置断点，然后单击绿色的调试图标开始调试。

如果你使用的是Facebook的[`create react app`](https://github.com/facebookincubator/create-react-app)，在Jest“运行/调试”配置的Jest package字段中指定`react-scripts`包的路径，并将`--env=jsdom` 添加到Jest options字段。

## 缓存问题

转换脚本已更改或Babel已更新，但这些更改没有被Jest识别？

使用[`--no cache`](CLI.md#--cache)重试。Jest缓存转换后的模块文件以加快测试执行，如果您正在使用自己的自定义转换器，请考虑向其中添加一个`getCacheKey` 函数：[getCacheKey in Relay](https://github.com/facebook/relay/blob/58cf36c73769690f0bbf90562707eadb062b029d/scripts/jest/preprocessor.js#L56-L61)。

## 未返回的Promises

如果一个promise并未返回任何东西（no resolve），你将看到类似如下错误：

```bash
- Error: Timeout - Async callback was not invoked within timeout specified by jasmine.DEFAULT_TIMEOUT_INTERVAL.`
```

通常情况下，这是由冲突的Promise实现所引起的。考虑使用你自己的promise实现替换全局的promise，例如：`global.Promise = jest.requireActual('promise');` 和/或 将使用过的promise库合并为单个库。

如果测试长时间运行，你可能需要考虑通过调用`jest.setTimeout`来增加超时阈值。

```js
jest.setTimeout(10000); // 10 second timeout
```

## 守卫问题

尝试在运行Jest时携带参数 [`--no-watchman`](CLI.md#--watchman) ，或者设置`watchman`配置选项为`false`。

更多详情，查看[守卫疑难排查](https://facebook.github.io/watchman/docs/troubleshooting)。

## 在Docker 和/或 持续集成（CI，Continuous Integration）服务器中运行est测试极慢

虽然Jest在现代多核电脑上运行的速度非常快，但在某些特殊设置上，Jest可能会很慢，就像我们的用户[已经](https://github.com/facebook/jest/issues/1395)[遇到的](https://github.com/facebook/jest/issues/1524#issuecomment-260246008)。

根据[调查结果](https://github.com/facebook/jest/issues/1524\issuecomment-262366820)所示，一种减轻此问题并将速度提高50%的方法是按顺序运行测试。

为此，可以使用[`--runInBand`](CLI.md#--runinband)参数在同一线程中运行测试：

```bash
# Using Jest CLI
jest --runInBand

# Using yarn test (e.g. with create-react-app)
yarn test --runInBand
```

在持续集成服务器（如Travis-CI）上加快测试执行时间的另一种方法是将max worker pool设置为~_4_。特别是在Travis-CI上，这可以将测试执行时间减少一半。注意：可用于开源项目的Travis CI _free_ 计划仅包含2个CPU内核。

```bash
# Using Jest CLI
jest --maxWorkers=4

# Using yarn test (e.g. with create-react-app)
yarn test --maxWorkers=4
```

## `coveragePathIgnorePatterns` 似乎没有起到作用

确保您没有额外使用 `babel-plugin-istanbul` 插件，Jest自身包装Istanbul，因此也规定了Istanbul要覆盖哪些文件。而使用`babel-plugin-istanbul`时，Babel处理的每个文件都将具有覆盖收集代码，因此`coveragePathIgnorePatterns`不会将其忽略。

## 定义测试

所有的测试必须被定义为同步性质，以便于Jest能够成功收集测试。

举一个例子来说明为什么要这样规定，假设我们编写了这样一个测试：

```js
// Don't do this it will not work
setTimeout(() => {
  it('passes', () => expect(1).toBe(1));
}, 0);
```

当Jest运行测试来收集`test`时，它将找不到任何内容，因为我们已经将定义设置为在事件循环的下一个周期中异步执行。

_注意_：这意味着当你使用 `test.each`方法时，你不能在`beforeEach` / `beforeAll`中异步设置表。

## 仍有疑问？

请查阅[帮助文档](/help.html)。
