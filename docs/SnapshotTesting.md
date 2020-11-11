---
id: snapshot-testing
title: Snapshot Testing
---

无论何时，如果你想确保自己的UI不会出现意外的变化，快照测试将是你非常有用的工具。

一个典型的移动端程序的快照测试用例包含如下过程：渲染一个UI组件、创建一个快照，然后将其与测试所存的参考快照文件进行比较。如果两份快照不能正确匹配，则表示测试不通过，其通常分为两种情况：一是UI组件产生了预期之外的改变，二是参考快照需要同步最新版本的UI组件。

## 使用Jest进行快照（Snapshot）测试

上述类似的方法同样能够应用于测试你的React组件。你可以使用一个测试渲染器为你的React tree生成可序列化的值，而不用渲染需要构建整个应用程序的图形化UI。可以参考一个针对[Link 组件](https://github.com/facebook/jest/blob/master/examples/snapshot/Link.react.js)的[示例测试](https://github.com/facebook/jest/blob/master/examples/snapshot/__tests__/link.react.test.js)：

```javascript
import React from 'react';
import Link from '../Link.react';
import renderer from 'react-test-renderer';

it('renders correctly', () => {
  const tree = renderer
    .create(<Link page="http://www.facebook.com">Facebook</Link>)
    .toJSON();
  expect(tree).toMatchSnapshot();
});
```

第一次运行该测试时，Jest会创建一个[快照文件](https://github.com/facebook/jest/blob/master/examples/snapshot/__tests__/__snapshots__/link.react.test.js.snap) ，如下所示：

```javascript
exports[`renders correctly 1`] = `
<a
  className="normal"
  href="http://www.facebook.com"
  onMouseEnter={[Function]}
  onMouseLeave={[Function]}
>
  Facebook
</a>
`;
```

快照文件应该同代码更改一起提交，并且作为你code review过程中的一个部分。Jest使用[pretty-format](https://github.com/facebook/jest/tree/master/packages/pretty-format) 在code review过程中使得快照代码具有可读性。在随后的测试运行过程中，Jest将组件所渲染的输出与先前的快照进行比较。如果两者匹配，则表示测试通过。如果两者不匹配，则说明测试运行器在你的代码（本例中为`<Link>`组件）中发现了一个bug，应予以修复，或者代码的实现已更改，对应的快照需要与之同步。

> 注意：快照直接作用于你渲染的数据————在我们的示例中，其为带有页面prop值传入的`<Link />`组件，这表示尽管其他任意文件（例如 `App.js`）使用`<Link />`组件时缺少props，它也能通过测试，因为该测试并不知道`<Link />`组件的实际用法，且测试范围仅限于`Link.react.js`文件。同样，在其他快照测试中使用不同的props来渲染同一组件并不会影响第一个测试，因为这些测试彼此之间并不相通

有关快照测试如何进行以及为什么构建快照测试的更多信息，请参考 [release blog post](https://jestjs.io/blog/2016/07/27/jest-14.html)。我们推荐你阅读[this blog post](http://benmccormick.org/2016/09/19/testing-with-jest-snapshots-first-impressions/)，以明确什么时候来使用快照测试。我们同样推荐你观看关于使用Jest进行快照测试的视频[egghead video](https://egghead.io/lessons/javascript-use-jest-s-snapshot-testing-feature?pl=testing-javascript-with-jest-a36c4074)。

### 更新快照

当代码中存在bug时，可以很容易在快照测试失败后进行定位。如果发生这种情况，请继续并修复对应问题，以确保快照测试再次通过。现在，让我们由于有意的更改组件实现而导致快照测试失败的情况。

如果我们有意更改示例中Link组件的链接地址的指向，则会出现这种情况。

```javascript
// Updated test case with a Link to a different address
it('renders correctly', () => {
  const tree = renderer
    .create(<Link page="http://www.instagram.com">Instagram</Link>)
    .toJSON();
  expect(tree).toMatchSnapshot();
});
```

在这种情况下。Jest将打印如下输出：

![](/img/content/failedSnapshotTest.png)

由于我们刚刚将组件更新为指向一个不同的链接地址，因此能够合理的预期此组件快照的更改。我们的快照测试示例测试未通过，原因在于已更改组件的快照不再与测试用例中的快照工件相匹配。

为解决此问题，我们将需要更新测试用例中的快照工件。你可以运行Jest，并附带一个标志参数，它表示重新生成快照。

```bash
jest --updateSnapshot
```

继续并通过运行上述命令接受更改。你也可以使用等效的单字符标志参数`-u`来重新生成快照。需要注意的是，这将为所有失败的快照测试重新生成快照工件，如果由于意外的bug而导致快照测试失败，我们需要在重新生成快照前，先修复该bug，以此避免记录带有错误行为的快照。

如果你想要限制重新生成哪些测试快照用例，则可以额外传递一个`--testNamePattern`参数标志来重新记录那些与模式相匹配的测试的快照。

你可以通过克隆[快照用例](https://github.com/facebook/jest/tree/master/examples/snapshot), 修改`Link`组件并运行Jest来测试这一功能。

### 交互性快照模式

失败的快照也能在监测模式下进行交互式更新：

![](/img/content/interactiveSnapshot.png)

进入交互性快照模式后，Jest将在一次测试中逐步引导你通过失败的快照，并给予你观察错误的日志输出。

此处，你可以选择更新此快照或者跳至下一步：

![](/img/content/interactiveSnapshotUpdate.gif)

完成上述过程后，Jest将在返回监测模式之前，为你提供一份简要报告。

![](/img/content/interactiveSnapshotDone.png)

### 内联快照

内联快照的行为与外部快照 (`.snap` 文件)基本一致，不同之处在于前者的快照值将自动写回源代码之中，这意味着你可以享受自动生成快照的好处，而不必切换到外部文件来确保写入正确的值。

> 内联快照由[Prettier](https://prettier.io)提供支持，使用内联快照时必须确保项目中已经安装`prettier`，当写入快照至测试文件时将遵循你的Prettier配置。
>
> 如果Jest无法找到你的`prettier`的安装位置，你可以通过设置[`"prettierPath"`](./Configuration.md#prettierpath-string) 配置属性来告诉Jest如何找到它.

**示例:**

First, you write a test, calling `.toMatchInlineSnapshot()` with no arguments:
首先，你写了一个不带参数的测试函数`.toMatchInlineSnapshot()`：

```javascript
it('renders correctly', () => {
  const tree = renderer
    .create(<Link page="https://prettier.io">Prettier</Link>)
    .toJSON();
  expect(tree).toMatchInlineSnapshot();
});
```

然后运行Jest时，`tree`将被计算出来，并且一份快照将被作为参数写入`toMatchInlineSnapshot`函数：

```javascript
it('renders correctly', () => {
  const tree = renderer
    .create(<Link page="https://prettier.io">Prettier</Link>)
    .toJSON();
  expect(tree).toMatchInlineSnapshot(`
<a
  className="normal"
  href="https://prettier.io"
  onMouseEnter={[Function]}
  onMouseLeave={[Function]}
>
  Prettier
</a>
`);
});
```

以上就是其全部了，你也可以通过运行时带上`--updateSnapshot`标志参数来更新快照，或者在`--watch`模式下输入`u`进行快照更新。

### 属性匹配器

通常，在对象中有一些字段需要快照，这些字段是运算生成的（比如id和Dates）。如果尝试对这些对象进行快照，它们将强制快照在每次运行时失败：

```javascript
it('will fail every time', () => {
  const user = {
    createdAt: new Date(),
    id: Math.floor(Math.random() * 20),
    name: 'LeBron James',
  };

  expect(user).toMatchSnapshot();
});

// Snapshot
exports[`will fail every time 1`] = `
Object {
  "createdAt": 2018-05-19T23:36:09.816Z,
  "id": 3,
  "name": "LeBron James",
}
`;
```

对于这些情况，Jest允许为任何属性提供非对称匹配器。在写入或测试快照之前，将检查这些匹配器，然后将其直接保存到快照文件而不是保存运算结果值：

```javascript
it('will check the matchers and pass', () => {
  const user = {
    createdAt: new Date(),
    id: Math.floor(Math.random() * 20),
    name: 'LeBron James',
  };

  expect(user).toMatchSnapshot({
    createdAt: expect.any(Date),
    id: expect.any(Number),
  });
});

// Snapshot
exports[`will check the matchers and pass 1`] = `
Object {
  "createdAt": Any<Date>,
  "id": Any<Number>,
  "name": "LeBron James",
}
`;
```

任何不是匹配器的给定值都将被精确检查并保存到快照：

```javascript
it('will check the values and pass', () => {
  const user = {
    createdAt: new Date(),
    name: 'Bond... James Bond',
  };

  expect(user).toMatchSnapshot({
    createdAt: expect.any(Date),
    name: 'Bond... James Bond',
  });
});

// Snapshot
exports[`will check the values and pass 1`] = `
Object {
  "createdAt": Any<Date>,
  "name": 'Bond... James Bond',
}
`;
```

## 最佳实践

快照是一个极好的工具，能用于识别应用程序中的意外接口更改，无论该接口是API响应、UI、日志还是错误消息。与任何测试策略一样，为了有效地使用它们，您应该了解一些最佳实践，并遵循一些指导原则。

### 1. 将快照视为代码

提交快照并将其作为常规代码检查过程的一部分进行检查，这意味着像对待项目中任何其他类型的测试文件或代码一样对待快照。

通过保持快照内容集中、简短，并使用一些强制约束这些风格的工具，来确保快照的可读性。

正如上文提到，Jest使用[`pretty-format`](https://yarnpkg.com/en/package/pretty-format)来保证快照的可读性，但你会发现额外引入其他工具非常有用，比如[`eslint-plugin-jest`](https://yarnpkg.com/en/package/eslint-plugin-jest)，通过配置它的[`no-large-snapshots`](https://github.com/jest-community/eslint-plugin-jest/blob/master/docs/rules/no-large-snapshots.md)选项，或者使用[`snapshot-diff`](https://yarnpkg.com/en/package/snapshot-diff)的组件快照比较功能，能够很好的促使你提交简短、集中的断言。

其目标是使你在代码PR（pull requests）中更容易地进行快照Code Review，并在测试套件失败时，促使你改变不检查其失败的根本原因而直接重新生成快照的坏习惯。

### 2. 测试应该是确定性的

你的测试应该是确定性的。在未更改的组件上多次运行相同的测试，每次都会产生相同的结果。你应该负责确保所生成的快照不包含特定于平台的数据或其他不确定的数据。

例如，现有一个[Clock](https://github.com/facebook/jest/blob/master/examples/snapshot/Clock.react.js)组件，其使用`Date.now()`方法，则每次运行测试用例时，此组件生成的快照都将不同。在这种情况下，我们可以[模拟（mock） Date.now() 方法](MockFunctions.md) ，以使得每次运行测试时保证返回值一致：

```js
Date.now = jest.fn(() => 1482363367071);
```

现在，每次快照测试用例运行时，`Date.now()`方法将始终返回一致结果`1482363367071`。这样，无论测试何时运行，都将确保为该组件生成相同的快照

### 3. 使用描述性快照名称

始终尽量去为快照文件使用具有描述性测试名称或快照名称。好的文件名称能够更详尽地描述快照的预期内容，从而使得审阅者更容易在代码检查过程中进行快照验证，并且在快照更新之前让所有人都明确过时的快照是否具有正确的行为。

例如，比较下面两段代码：

```js
exports[`<UserName /> should handle some test case`] = `null`;

exports[`<UserName /> should handle some other test case`] = `
<div>
  Alan Turing
</div>
`;
```

To:

```js
exports[`<UserName /> should render null`] = `null`;

exports[`<UserName /> should render Alan Turing`] = `
<div>
  Alan Turing
</div>
`;
```

Since the later describes exactly what's expected in the output, it's more clear to see when it's wrong:
由于后者准确地描述了输出中预期的内容，因此更清楚地看到错误：

```js
exports[`<UserName /> should render null`] = `
<div>
  Alan Turing
</div>
`;

exports[`<UserName /> should render Alan Turing`] = `null`;
```

## 常见问题解答

### 快照是否自动写入连续集成（Continuous Integration，CI）系统？

并不是。从Jest 20开始，当Jest在CI系统中运行而没有显式传递`--updateSnapshot`时，Jest中的快照不会自动写入其中。所有快照都是在CI上运行的代码的一部分，由于新快照会自动通过，因此它们不应在CI系统上的测试运行。此外，建议始终提交所有快照并将其保存在版本控制之中。

### 是否应提交快照文件？

是的。所有快照文件都应该与它们所覆盖的模块及其测试一起提交。它们应该被视为测试的一部分，类似于任何其他在Jest中断言的值。实际上，快照表示源模块在任何给定时间点的状态。这样，当源模块被修改时，Jest可以判断出与前一个版本相比发生了什么变化。它还可以在Code Review期间提供许多额外的上下文信息，在这些上下文信息中审阅者可以更好地研究你的更改。

### 快照测试只适用于React组件吗？

[React](TutorialReact.md) and [React Native](TutorialReactNative.md) 组件是快照测试的一个很好的用例。事实上，快照可以捕获任何可序列化的值，并且应在任何需要判断目标测试输出是否正确的场景下使用。Jest仓库包含许多测试Jest本身输出、Jest断言库的输出以及来自Jest代码库各个部分的日志消息的示例。请参见Jest仓库中[snapshotting CLI output](https://github.com/facebook/jest/blob/master/e2e/__tests__/console.test.ts)的示例。

### 快照测试和可视化回归测试有什么区别？

快照测试和可视化回归测试是测试UI的两种不同的方法，它们的目的不同。可视化回归测试工具获取网页的屏幕截图，并逐像素比较结果图像。使用快照测试时，组件值被序列化，存储在文本文件中，并使用diff算法进行比较。有许多不同的权衡项需要被考虑，我们在[Jest blog](https://jestjs.io/blog/2016/07/27/jest-14.html#why-snapshot-testing)中列出了构建快照测试的原因。

### 快照测试能取代单元测试吗？

快照测试只是Jest附带的20多个断言工具中的一个。快照测试的目的不是取代现有的单元测试，而是提供附加选项，以使测试变得轻松。在某些场景中，快照测试可能会移除对特定功能集（例如React组件）的单元测试需求，但它们也可以一起工作。

### 关于快照测试的性能，以及其生成文件的速度和大小如何？

Jest已经在代码重构时考虑到了性能，其中的快照测试也不例外。由于快照存储在文本文件中，因此这种测试方法既快速又可靠。Jest为调用 `toMatchSnapshot`匹配器的每个测试文件生成一个新文件。快照的大小非常小：作为参考，Jest代码库中所有快照文件的大小都小于300kb。

### 如何解决快照文件中的冲突？

快照文件必须始终表示它们所覆盖的模块的当前状态。因此，当合并两个分支并在快照文件中遇到冲突，则可以手动解决冲突，也可以通过运行Jest并检查结果来更新快照文件。

### 是否可以将测试驱动开发原则应用于快照测试？

虽然可以手动写入快照文件，但这通常是不可实现的。快照有助于确定测试覆盖的模块的输出是否发生了更改，而不是首先为代码的设计提供指导。

### 代码覆盖能够和快照测试一起使用吗？

是的，其能够与其他任何测试一起使用
