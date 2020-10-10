---
id: migration-guide
title: 迁移至 Jest
---

如果你想在现有的代码库中试用 Jest，有很多方法：

- 如果你用的是 Jasmine 或者类似 Jasmine 的 API（例如 [Mocha](https://mochajs.org)），Jest 在大多数情况下应该都是兼容的，这使得迁移到 Jest 更简单。
- 如果你用的是 AVA、Expect.js（ 由 Automattic 提供）、Jasmine、Mocha、proxyquire、Should.js 或 Tape，则可以使用 Jest Codemods 自动迁移（请参见下文）。
- 如果你喜欢 [chai](http://chaijs.com/)，则可以升级到 Jest 并继续使用 chai。但是，我们建议尝试 Jest 的断言及其失败的提示信息。Jest Codemods 可以从 chai 迁移（请参见下文）。

## jest-codemods

如果你使用的是 [AVA](https://github.com/avajs/ava)、[Chai](https://github.com/chaijs/chai)、[Expect.js (by Automattic)](https://github.com/Automattic/expect.js)、[Jasmine](https://github.com/jasmine/jasmine)、[Mocha](https://github.com/mochajs/mocha)、[proxyquire](https://github.com/thlorenz/proxyquire)、 [Should.js](https://github.com/shouldjs/should.js) 或者 [Tape](https://github.com/substack/tape)， 那么你可以用第三方的 [jest-codemods](https://github.com/skovhus/jest-codemods) 来完成大部分麻烦的迁移工作。它使用 [jscodeshift](https://github.com/facebook/jscodeshift) 在你的代码库上实现代码转换。

要将现有的测试转换成 Jest，需要在包含测试的项目中运行：

```bash
npx jest-codemods
```

更多信息请参见 [https://github.com/skovhus/jest-codemods](https://github.com/skovhus/jest-codemods)。
