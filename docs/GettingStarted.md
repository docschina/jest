---
id: getting-started
title: 快速入门
---

使用 [`yarn`](https://yarnpkg.com/en/package/jest) 安装 Jest：

```bash
yarn add --dev jest
```

或使用 [`npm`](https://www.npmjs.com/) 安装：

```bash
npm install --save-dev jest
```

注：Jest 的文档统一使用 `yarn` 指令，但使用 `npm` 同样可行。可以通过[ yarn 官方文档](https://yarnpkg.com/en/docs/migrating-from-npm#toc-cli-commands-comparison)进行 `yarn` 和 `npm` 的对比。

下面我们开始给一个假定的函数写测试，这个函数的功能是两数相加。首先创建 `sum.js` 文件：

```javascript
function sum(a, b) {
  return a + b;
}
module.exports = sum;
```

接下来，创建名为 `sum.test.js` 的文件。这个文件包含了实际测试内容：

```javascript
const sum = require('./sum');

test('adds 1 + 2 to equal 3', () => {
  expect(sum(1, 2)).toBe(3);
});
```

将如下代码添加到 `package.json` 中。

```json
{
  "scripts": {
    "test": "jest"
  }
}
```

最后，运行 `yarn test` 或者 `npm run test` ，Jest 将输出如下信息：

```bash
PASS  ./sum.test.js
✓ adds 1 + 2 to equal 3 (5ms)
```

**你刚才使用 Jest 成功地写出了第一个测试！**

在此测试中，使用了 `expect` 和 `toBe` 来检测两个值是否完全相同。若要了解其它使用 Jest 可以测试的内容，请参阅 [使用匹配器(Matcher)](UsingMatchers.md)。

## 使用命令行

你可以直接从 CLI 运行 Jest 并使用一系列有用的配置参数（前提是已经配置了环境变量 `PATH` 使其全局可用，例如通过 `yarn global add jest` 或者 `npm install jest --global` 来安装 Jest）。

这里演示了如何对于能够匹配到 `my-test` 的文件运行 Jest，以 `config.json` 作为配置文件，并在运行后显示一个基于原生操作系统的通知：

```bash
jest my-test --notify --config=config.json
```

如果想要了解更多在命令行中运行 `jest` 的内容，请参阅 [Jest CLI 配置项](CLI.md)页面.

## 更多配置项

### 生成基础配置文件

Jest 将根据你的项目提出一系列问题，并且将创建一个基础配置文件。文件中的每一项都配有简短的说明：

```bash
jest --init
```

### 使用 Babel

通过 `yarn` 安装所需要的依赖来使用 [Babel](http://babeljs.io/)。

```bash
yarn add --dev babel-jest @babel/core @babel/preset-env
```

在项目的根目录下创建 `babel.config.js` ，通过配置 Babel 使其能够兼容当前的 Node 版本。

```javascript
// babel.config.js
module.exports = {
  presets: [
    [
      '@babel/preset-env',
      {
        targets: {
          node: 'current',
        },
      },
    ],
  ],
};
```

**Babel 的适配取决于项目本身。** 查阅 [Babel 官方文档](https://babeljs.io/docs/en/)来获得更多详细信息。

<details><summary markdown="span"><strong>将 Babel 配置为 Jest 可感知的</strong></summary>

如果没有被设置成其它值，Jest 会把 `process.env.NODE_ENV` 设置成 `'test'` 。你可以运用在配置项中，从而根据实际情况设定适用于 Jest 的编译。例如：

```javascript
// babel.config.js
module.exports = api => {
  const isTest = api.env('test');
  // 你可以使用 isTest 来决定需要使用到的预设和插件。

  return {
    // ...
  };
};
```

> 注：`babel-jest` 是在安装 Jest 时自动安装的，如果在你的项目中有 babel 的配置内容，那么它会自动进行文件的转换。如果想要避免这一行为，你可以显式地重新配置 `transform` 配置项：

```javascript
// jest.config.js
module.exports = {
  transform: {},
};
```

</details>

<details><summary markdown="span"><strong>Babel 6 支持</strong></summary>

Jest 24 不再支持 Babel 6。我们强烈推荐升级至目前正在积极维护的 Babel 7。但是如果你无法升级到 Babel 7，可以继续使用 Jest 23，或者升级到 Jest 24 并且使 `babel-jest` 固定在 23 的版本，如下面的例子所示：

```
"dependencies": {
  "babel-core": "^6.26.3",
  "babel-jest": "^23.6.0",
  "babel-preset-env": "^1.7.0",
  "jest": "^24.0.0"
}
```

虽然对于每一个 Jest 包来说，我们普遍推荐使用相同版本的 Babel，但是上述方式目前可以让你在使用最新的 Jest 的版本的同时使用 Babel 6。

</details>

### 使用 webpack

Jest 可以用于在使用 [webpack](https://webpack.js.org/) 管理资源、样式和编译方式的项目中。webpack 和其它工具相比的确多了一些独特的挑战。请参阅 [webpack 指南](Webpack.md) 快速上手。

### 使用 parcel

Jest 可以用于在使用 [parcel-bundler](https://parceljs.org/) 管理资源、样式和编译方式的项目中，和 webpack 类似。Parcel 不需要任何配置。请参考官方[文档](https://parceljs.org/getting_started.html)快速入门。

### 使用 Typescript

通过 Babel，Jest 能够支持 Typescript。首先要确保你遵循了上述[使用 Babel](#using-babel) 指引。接下来使用 `yarn` 安装 `@babel/preset-typescript` ：

```bash
yarn add --dev @babel/preset-typescript
```

然后将 `@babel/preset-typescript` 添加到 `babel.config.js` 中的 presets 列表中。

```diff
// babel.config.js
module.exports = {
  presets: [
    ['@babel/preset-env', {targets: {node: 'current'}}],
+    '@babel/preset-typescript',
  ],
};
```

不过，在使用 Typescript 配合 Babel 的过程中，有一些[注意事项](https://babeljs.io/docs/en/next/babel-plugin-transform-typescript.html#caveats)。由于 Babel 是通过代码语言转换(Transpilation)支持 Typescript 的，Jest 在运行测试的过程中不会进行类型检查。如果想要此功能，可以使用 [ts-jest](https://github.com/kulshekhar/ts-jest)。