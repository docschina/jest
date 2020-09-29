---
id: manual-mocks
title: 手动模拟
---

手动模拟用于模拟数据存入功能。例如，你可能不希望访问网站或数据库之类的远程资源，而是想创建一个允许使用假数据的手动模拟。这样可以确保你在测试时快速且不易出错。

## 模拟 user 模块

手动模拟是通过在紧邻模块的 `__mocks__/` 子目录中写入模块来定义的。例如，要在 `models` 目录中模拟一个名为 `user` 的模块，需要创建一个名为 `user.js` 的文件，然后将该文件放入 `models/__mocks__` 目录中。注意 `__mocks__` 文件夹区分大小写，如果写成 `__MOCKS__`，在某些系统上测试可能会中断。

> 当测试中需要某个模块时，**必须**显式调用 `jest.mock('./moduleName')`。

## 模拟 Node 模块

如果模拟一个 Node 模块（例如：`lodash` ），那么模拟的代码文件应该放在与 `node_modules` 相邻的 `__mocks__` 目录中（除非将 [`根目录`](Configuration.md#roots-arraystring) 配置为项目根目录以外的文件夹），此时模拟将**自动**执行。没必要显示调用 `jest.mock('module_name')`。

可以通过在目录结构中创建一个与作用域模块名称匹配的文件来模拟作用域模块。例如，模拟一个名为 `@scope/project-name` 的作用域模块，需要创建一个 `__mocks__/@scope/project-name.js` 文件，这会相应地创建 `@scope/` 目录。

> 请注意： 如果想模拟 Node 的核心模块（例如：`fs` 或 `path`），那么**需要**显示调用该模块，例如 `jest.mock('path')`，因为默认情况下不会对 Node 的核心模块进行模拟。

## 实例演示

```bash
.
├── config
├── __mocks__
│   └── fs.js
├── models
│   ├── __mocks__
│   │   └── user.js
│   └── user.js
├── node_modules
└── views
```

当给定模块存在手动模拟时，Jest 的模块系统将在显示调用 `jest.mock('moduleName')` 时使用该模块。但是，当 `automock` 设置为 `true` 时， 即使未调用 `jest.mock('moduleName')`，也会使用手动模拟实现，而不是用自动创建的模拟。要是不想用这种方式，需要在应该使用实际模块实现的测试中显式调用 `jest.unmock('moduleName')`。

> 注意： 为了正确模拟，Jest 需要让 `jest.mock('moduleName')` 语句和 `require/import` 语句处于同一作用域中。

下面的代码是一个手写的实例，代码中有一个模块，可以提供给定目录中的所有文件。这种情况下使用（内置的）核心模块 `fs` 。

```javascript
// FileSummarizer.js
'use strict';

const fs = require('fs');

function summarizeFilesInDirectorySync(directory) {
  return fs.readdirSync(directory).map(fileName => ({
    directory,
    fileName,
  }));
}

exports.summarizeFilesInDirectorySync = summarizeFilesInDirectorySync;
```

由于我们希望我们的测试避免实际操作磁盘（这样做很慢且不稳定），因此通过扩展自动模拟为 `fs` 模块来创建手动模拟。下面代码中的手动模拟将实现可用于测试的 `fs` API 的自定义版本：

```javascript
// __mocks__/fs.js
'use strict';

const path = require('path');

const fs = jest.createMockFromModule('fs');

// 这是一个自定义函数，我们的测试可以在安装过程中使用它来指定
// 使用任何 `fs` API 时
// “模拟”文件系统上的文件内容
let mockFiles = Object.create(null);
function __setMockFiles(newMockFiles) {
  mockFiles = Object.create(null);
  for (const file in newMockFiles) {
    const dir = path.dirname(file);

    if (!mockFiles[dir]) {
      mockFiles[dir] = [];
    }
    mockFiles[dir].push(path.basename(file));
  }
}

// 自定义版本的 `readdirSync` 函数，它从
// __setMockFiles 设置的特殊模拟文件列表中读取文件
function readdirSync(directoryPath) {
  return mockFiles[directoryPath] || [];
}

fs.__setMockFiles = __setMockFiles;
fs.readdirSync = readdirSync;

module.exports = fs;
```

现在我们编写测试代码。请注意，由于 `fs` 模块是 Node 的核心模块，因此需要明确告知我们要模拟该模块：

```javascript
// __tests__/FileSummarizer-test.js
'use strict';

jest.mock('fs');

describe('listFilesInDirectorySync', () => {
  const MOCK_FILE_INFO = {
    '/path/to/file1.js': 'console.log("file1 contents");',
    '/path/to/file2.txt': 'file2 contents',
  };

  beforeEach(() => {
    // 在每次测试之前设置一些模拟文件信息
    require('fs').__setMockFiles(MOCK_FILE_INFO);
  });

  test('includes all files in the directory in the summary', () => {
    const FileSummarizer = require('../FileSummarizer');
    const fileSummary = FileSummarizer.summarizeFilesInDirectorySync(
      '/path/to',
    );

    expect(fileSummary.length).toBe(2);
  });
});
```

这个例子使用 [`jest.createMockFromModule`](JestObjectAPI.md#jestcreatemockfrommodulemodulename) 生成自动模拟，并覆盖其默认行为。这是推荐的方法，但完全是可选的。如果你根本不想用自动模拟，则只需从模拟文件中导出自己的函数即可。完全手动模拟的一个缺点在于，它们是手动的——这意味着你必须在它们模拟的模块发生变化时手动更新。因此，最好在满足需求的时候使用或者扩展自动模拟。

为了确保手动模拟和它的实际实现保持同步，可能需要在手动模拟中使用 [`jest.requireActual(moduleName)`](JestObjectAPI.md#jestrequireactualmodulename) 来引用实际模块，并在导出之前使用模拟函数修改它。

可以在 [examples/manual-mocks](https://github.com/facebook/jest/tree/master/examples/manual-mocks) 上找到该示例的代码。

## 与 ES 导入模块配合使用

如果你使用 [ES 的导入模块](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import)，那么通常会将 `import` 语句放在测试文件的顶部。但是通常你需要在模块使用之前让 Jest 使用模拟。因此，Jest 会自动将 `jest.mock` 调用提升到模块的顶部（在任何 import 语句之前）。要了解更多相关信息并进行实际操作，请参见 [this repo](https://github.com/kentcdodds/how-jest-mocking-works).

## JSDOM 中未实现的模拟方法

如果某些代码使用的方法在 JSDOM（Jest 使用的 DOM 实现）中还未实现，那么测试起来可能并不容易。例如，`window.matchMedia()`，执行时 Jest 会返回 `TypeError: window.matchMedia is not a function`，且无法正确执行测试。

在这种情况下，在测试文件中模拟 `matchMedia` 应该可以解决这个问题：

```js
Object.defineProperty(window, 'matchMedia', {
  writable: true,
  value: jest.fn().mockImplementation(query => ({
    matches: false,
    media: query,
    onchange: null,
    addListener: jest.fn(), // 不推荐使用
    removeListener: jest.fn(), // 不推荐使用
    addEventListener: jest.fn(),
    removeEventListener: jest.fn(),
    dispatchEvent: jest.fn(),
  })),
});
```

如果在测试中调用的函数（或方法）中使用了 `window.matchMedia()`，这是可以的。如果在测试文件中直接执行 `window.matchMedia()`，Jest 会报相同的错误提示。在这种情况下，解决方案是将手动模拟放到一个单独的文件中，并在测试文件**之前**将其包含在测试中：

```js
import './matchMedia.mock'; // 必须在测试文件之前导入
import {myMethod} from './file-to-test';

describe('myMethod()', () => {
  // 在这里测试方法...
});
```
