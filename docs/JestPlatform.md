---
id: jest-platform
title: Jest 平台
---

你可以使用 cherry pick 拉取 Jest 中的特定功能，将其作为独立的包进行引用。下面列出了可用的包：

## jest-changed-files

一个用于在 git 或 hg 仓库中识别发生修改文件的工具。导出了两个函数：

- `getChangedFilesForRoots` 返回一个 promise 对象，resolve 的内容是一个包含了发生了修改的文件和仓库的 object。
- `findRepos` 返回一个 promise 对象，resolve 的内容是一系列指定路径中包含的仓库。

### 举例

```javascript
const {getChangedFilesForRoots} = require('jest-changed-files');

// 打印出当前仓库下最后一次 commit 之后修改过的文件
getChangedFilesForRoots(['./'], {
  lastCommit: true,
}).then(result => console.log(result.changedFiles));
```

你可以在 [readme 文件](https://github.com/facebook/jest/blob/master/packages/jest-changed-files/README.md)中了解 `jest-changed-files` 的更多内容。

## jest-diff

一个用于数据变化可视化的工具。导出了一个函数，这个函数用于比较任意类型的两个值，并返回一个“美化了打印”的字符串，用于展示两个参数之间的差异。

### 举例

```javascript
const diff = require('jest-diff').default;

const a = {a: {b: {c: 5}}};
const b = {a: {b: {c: 6}}};

const result = diff(a, b);

// 打印出差异
console.log(result);
```

## jest-docblock

一个用于提取和转换 JavaScript 文件中顶部注释的工具。导出多个函数对注释块中的数据进行操作。

### 举例

```javascript
const {parseWithComments} = require('jest-docblock');

const code = `
/**
 * 这是一个样例
 *
 * @flow
 */

 console.log('Hello World!');
`;

const parsed = parseWithComments(code);

// 打印出一个 object，它具有两个属性: comments 和 pragmas。
console.log(parsed);
```

你可以在 [readme 文件](https://github.com/facebook/jest/blob/master/packages/jest-docblock/README.md)中了解 `jest-docblock` 的更多内容。

## jest-get-type

一个能够识别任何属于 JavaScript 值的原始类型的模块。导出了一个函数，这个函数返回一个字符串，表示作为参数传入的值的类型。

### 举例

```javascript
const getType = require('jest-get-type');

const array = [1, 2, 3];
const nullValue = null;
const undefinedValue = undefined;

// 打印出 'array'
console.log(getType(array));
// 打印出 'null'
console.log(getType(nullValue));
// 打印出 'undefined'
console.log(getType(undefinedValue));
```

## jest-validate

一个用于校验用户所提交的配置项的工具。导出一个函数，这个函数需要两个参数：用户的配置项和一个包含了配置项示例与其它选项的 object。返回值是一个有两个属性的 object：

- `hasDeprecationWarnings`，一个布尔值，表示所提交的配置项是否有不推荐的警告(Deprecation Warnings),
- `isValid`，一个布尔值，表示配置项是否正确。

### 举例

```javascript
const {validate} = require('jest-validate');

const configByUser = {
  transform: '<rootDir>/node_modules/my-custom-transform',
};

const result = validate(configByUser, {
  comment: '  Documentation: http://custom-docs.com',
  exampleConfig: {transform: '<rootDir>/node_modules/babel-jest'},
});

console.log(result);
```

你可以在 [readme 文件](https://github.com/facebook/jest/blob/master/packages/jest-validate/README.md)中了解 `jest-validate` 的更多内容。

## jest-worker

一个用于任务之间平行化(Parallelization)的模块。导出了一个 `JestWorker` 类，其中包含了 Node.js 的模块路径，如果模块导出的方法是类级方法，则可以允许调用，并且返回一个 promise，在指定的方法于一个分叉进程(Forked Process)中结束执行时 resolve。

### Example

```javascript
// heavy-task.js

module.exports = {
  myHeavyTask: args => {
    // 长时间运行的 CPU 密集型任务.
  },
};
```

```javascript
// main.js

async function main() {
  const worker = new Worker(require.resolve('./heavy-task.js'));

  // 使用不同参数，并行运行两个任务
  const results = await Promise.all([
    worker.myHeavyTask({foo: 'bar'}),
    worker.myHeavyTask({bar: 'foo'}),
  ]);

  console.log(results);
}

main();
```

你可以在 [readme 文件](https://github.com/facebook/jest/blob/master/packages/jest-worker/README.md)中了解 `jest-worker` 的更多内容。

## pretty-format

导出一个函数，这个函数能够将任何属于 JavaScript 的值转换成人类可读的字符串。它支持所有 JavaScript 的内置类型，开箱即用，并且允许通过用户定义的插件来对应用程序中特定的类型进行扩展。

### 举例

```javascript
const prettyFormat = require('pretty-format');

const val = {object: {}};
val.circularReference = val;
val[Symbol('foo')] = 'foo';
val.map = new Map([['prop', 'value']]);
val.array = [-0, Infinity, NaN];

console.log(prettyFormat(val));
```

你可以在 [readme 文件](https://github.com/facebook/jest/blob/master/packages/pretty-format/README.md)中了解 `pretty-format` 的更多内容。
