---
id: puppeteer
title: 结合 Puppeteer 使用
---

通过[全局配置/卸载](Configuration.md#globalsetup-string)和[异步测试环境](Configuration.md#testenvironment-string)的 API, Jest 就可以与 [puppeteer](https://github.com/GoogleChrome/puppeteer) 无缝衔接了。

> 目前，如果你的测试中使用了 `page.$eval`，`page.$$eval` 或者 `page.evaluate`，那么就无法使用 Puppeteer 给测试文件生成覆盖代码。这是因为传入的函数是在 Jest 的作用域以外执行的。请在 Github 上的 [issue #7962](https://github.com/facebook/jest/issues/7962#issuecomment-495272339) 中查看应对措施。

## 使用 jest-puppeteer 预设

[Jest Puppeteer](https://github.com/smooth-code/jest-puppeteer) 提供了使用 Puppeteer 运行测试代码所需要的所有配置。

1. 首先，安装 `jest-puppeteer`

```
yarn add --dev jest-puppeteer
```

2. 在 Jest 的配置项中指定预设:

```json
{
  "preset": "jest-puppeteer"
}
```

3. 编写测试代码

```js
describe('Google', () => {
  beforeAll(async () => {
    await page.goto('https://google.com');
  });

  it('should be titled "Google"', async () => {
    await expect(page.title()).resolves.toMatch('Google');
  });
});
```

这里是不需要载入任何依赖的。Puppeteer 的 `page` 和 `browser` 类会自动暴露出来。

请查阅[相关文档](https://github.com/smooth-code/jest-puppeteer)。

## 不使用 jest-puppeteer 预设的自定义示例

你也可以从零开始自行接入 puppeteer。基本思路是：

1. 在全局配置中启动并写入 puppeteer 的 websocket 端点；
2. 从每个测试环境中连接到 puppeteer；
3. 通过全局卸载关闭 puppeteer。

下面是一个全局配置的例子：

```js
// setup.js
const puppeteer = require('puppeteer');
const mkdirp = require('mkdirp');
const path = require('path');
const fs = require('fs');
const os = require('os');

const DIR = path.join(os.tmpdir(), 'jest_puppeteer_global_setup');

module.exports = async function () {
  const browser = await puppeteer.launch();
  // 储存浏览器示例，以便于之后卸载
  // 这里的全局变量只会在卸载时允许访问，测试环境中无法访问
  global.__BROWSER_GLOBAL__ = browser;

  // 使用文件系统给测试环境暴露出 wsEndpoint
  mkdirp.sync(DIR);
  fs.writeFileSync(path.join(DIR, 'wsEndpoint'), browser.wsEndpoint());
};
```

接下来我们需要一个自定义的 puppeteer 测试环境

```js
// puppeteer_environment.js
const NodeEnvironment = require('jest-environment-node');
const fs = require('fs');
const path = require('path');
const puppeteer = require('puppeteer');
const os = require('os');

const DIR = path.join(os.tmpdir(), 'jest_puppeteer_global_setup');

class PuppeteerEnvironment extends NodeEnvironment {
  constructor(config) {
    super(config);
  }

  async setup() {
    await super.setup();
    // 获取到 wsEndpoint
    const wsEndpoint = fs.readFileSync(path.join(DIR, 'wsEndpoint'), 'utf8');
    if (!wsEndpoint) {
      throw new Error('wsEndpoint not found');
    }

    // 连接上 puppeteer
    this.global.__BROWSER__ = await puppeteer.connect({
      browserWSEndpoint: wsEndpoint,
    });
  }

  async teardown() {
    await super.teardown();
  }

  runScript(script) {
    return super.runScript(script);
  }
}

module.exports = PuppeteerEnvironment;
```

最后我们可以关闭 puppeteer 实例，并且清理掉文件。

```js
// teardown.js
const os = require('os');
const rimraf = require('rimraf');
const path = require('path');

const DIR = path.join(os.tmpdir(), 'jest_puppeteer_global_setup');
module.exports = async function () {
  // 关闭浏览器实例
  await global.__BROWSER_GLOBAL__.close();

  // 清理 wsEndpoint 文件
  rimraf.sync(DIR);
};
```

以上都准备好之后，我们现在就可以编写我们的测试代码了，如下所示：

```js
// test.js
const timeout = 5000;

describe(
  '/ (Home Page)',
  () => {
    let page;
    beforeAll(async () => {
      page = await global.__BROWSER__.newPage();
      await page.goto('https://google.com');
    }, timeout);

    it('should load without error', async () => {
      const text = await page.evaluate(() => document.body.textContent);
      expect(text).toContain('google');
    });
  },
  timeout,
);
```

最后，设置 `jest.config.js` 来从这些文件中读取内容。(`jest-puppeteer` 预设的幕后其实就是在做类似的事情)

```js
module.exports = {
  globalSetup: './setup.js',
  globalTeardown: './teardown.js',
  testEnvironment: './puppeteer_environment.js',
};
```

这里是[完整功能示例](https://github.com/xfumihiro/jest-puppeteer-example)的代码。
