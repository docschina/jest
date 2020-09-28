
# 使用 webpack

Jest 可以用在使用了 [webpack](https://webpack.js.org/) 的项目中去管理静态资源、样式和编译。与其它工具相比 webpack 有一些独特的优势，因为它直接与应用程序集成，允许管理样式表、图像和字体等资源，并且对于编译 Javascript 代码， Webpack 提供了广阔的生态系统和工具。

## 一个 webpack 例子

让我们从一种常见的webpack配置文件开始，并将其转换为Jest设置。

```js
// webpack.config.js
module.exports = {
  module: {
    loaders: [
      {exclude: ['node_modules'], loader: 'babel', test: /\.jsx?$/},
      {loader: 'style-loader!css-loader', test: /\.css$/},
      {loader: 'url-loader', test: /\.gif$/},
      {loader: 'file-loader', test: /\.(ttf|eot|svg)$/},
    ],
  },
  resolve: {
    alias: {
      config$: './configs/app-config.js',
      react: './vendor/react-master',
    },
    extensions: ['', 'js', 'jsx'],
    modules: [
      'node_modules',
      'bower_components',
      'shared',
      '/shared/vendor/modules',
    ],
  },
};
```

如果您有通过 Babel 转换的JavaScript文件，你可以通过安装 `babel-jest` 插件来 [启用 Babel](GettingStarted.md#using-babel) 。非 Babel 转换的JavaScript 文件可以使用 Jest 的 [`transform`](Configuration.md#transform-objectstring-pathtotransformer--pathtotransformer-object) 配置项来处理。

### 处理静态资源

接下来，让我们配置 Jest 来优雅地处理一些资源，例如样式文件和图像。 通常，这些文件在测试中并不是特别有用，因此我们可以安全地将它们模拟出来。 但是，如果您使用了CSS Modules，那么最好模拟一个代理来查找类名。

```json
// package.json
{
  "jest": {
    "moduleNameMapper": {
      "\\.(jpg|jpeg|png|gif|eot|otf|webp|svg|ttf|woff|woff2|mp4|webm|wav|mp3|m4a|aac|oga)$": "<rootDir>/__mocks__/fileMock.js",
      "\\.(css|less)$": "<rootDir>/__mocks__/styleMock.js"
    }
  }
}
```

mock 文件如下：

```js
// __mocks__/styleMock.js

module.exports = {};
```

```js
// __mocks__/fileMock.js

module.exports = 'test-file-stub';
```

### 模拟 CSS Modules

你可以用 [ES6 代理](https://github.com/keyanzhang/identity-obj-proxy) 来模拟 [CSS Modules](https://github.com/css-modules/css-modules):

```bash
yarn add --dev identity-obj-proxy
```

然后在 styles 对象里的的所有类名都将原样返回 (例： `styles.foobar === 'foobar'`). 这对于 React 的 [快照测试](SnapshotTesting.md) 是非常方便的.

```json
// package.json (for CSS Modules)
{
  "jest": {
    "moduleNameMapper": {
      "\\.(jpg|jpeg|png|gif|eot|otf|webp|svg|ttf|woff|woff2|mp4|webm|wav|mp3|m4a|aac|oga)$": "<rootDir>/__mocks__/fileMock.js",
      "\\.(css|less)$": "identity-obj-proxy"
    }
  }
}
```

> 注意，在 Node 6 中代理是默认启用的，如果当前不是在 Node 6 环境下, 请确保使用 `node --harmony_proxies node_modules/.bin/jest`来调用 Jest.

如果 `moduleNameMapper` 不足以满足你的需求, 你可以使用 Jest 的 [`transform`](Configuration.md#transform-objectstring-pathtotransformer--pathtotransformer-object) 配置项来指定资源是如何转化的。 举个列子, 一个返回文件基名的转换器 (比如 `require('logo.jpg');` 返回 `'logo'`) 可以写成:

```js
// fileTransformer.js
const path = require('path');

module.exports = {
  process(src, filename, config, options) {
    return 'module.exports = ' + JSON.stringify(path.basename(filename)) + ';';
  },
};
```

```json
// package.json (用于指定转换器和 CSS Modules)
{
  "jest": {
    "moduleNameMapper": {
      "\\.(css|less)$": "identity-obj-proxy"
    },
    "transform": {
      "\\.(jpg|jpeg|png|gif|eot|otf|webp|svg|ttf|woff|woff2|mp4|webm|wav|mp3|m4a|aac|oga)$": "<rootDir>/fileTransformer.js"
    }
  }
}
```

我们告诉 Jest 去忽略与样式表或图像扩展名匹配的文件，而需要得是我们的 mock 文件。你可以调整正则表达式来匹配你的 webpack 配置处理的文件类型。。

_注意: 如果您将 babel jest和其他代码预处理器一起使用，则必须显式地将 babel jest 定义为 JavaScript 代码的转换器，以便将“.js”文件映射到 babel jest 模块._

```json
"transform": {
  "^.+\\.js$": "babel-jest",
  "^.+\\.css$": "custom-transformer",
  ...
}
```

### 配置 Jest 来查找文件

既然 Jest 知道如何处理我们的文件, 我们需要告诉它如何去查找它们. 对于 webpack 的 `modulesDirectories`, 和 `extensions` 选项， Jest 的 `moduleDirectories` 和 `moduleFileExtensions` 与其有直接的相似之处.

```json
// package.json
{
  "jest": {
    "moduleFileExtensions": ["js", "jsx"],
    "moduleDirectories": ["node_modules", "bower_components", "shared"],

    "moduleNameMapper": {
      "\\.(css|less)$": "<rootDir>/__mocks__/styleMock.js",
      "\\.(gif|ttf|eot|svg)$": "<rootDir>/__mocks__/fileMock.js"
    }
  }
}
```

> 注意: `<rootDir>` 是一个特殊的标记，是 Jest 用你项目的根目录替换而来. 大部分情况下会在 `package.json` 文件所在的文件夹下， 除非你在配置中指定了自定义的“rootDir”选项.

与 webpack 的 `resolve.root` 选项功能类似，你也可以设置 `NODE_PATH` 环境变量, 或者使用 `modulePaths` 选项。

```json
// package.json
{
  "jest": {
    "modulePaths": ["/shared/vendor/modules"],
    "moduleFileExtensions": ["js", "jsx"],
    "moduleDirectories": ["node_modules", "bower_components", "shared"],
    "moduleNameMapper": {
      "\\.(css|less)$": "<rootDir>/__mocks__/styleMock.js",
      "\\.(gif|ttf|eot|svg)$": "<rootDir>/__mocks__/fileMock.js"
    }
  }
}
```

最后，为了我们可以再次使用“moduleNameMapper”选项，我们要处理 webpack 的“alias”。

```json
// package.json
{
  "jest": {
    "modulePaths": ["/shared/vendor/modules"],
    "moduleFileExtensions": ["js", "jsx"],
    "moduleDirectories": ["node_modules", "bower_components", "shared"],

    "moduleNameMapper": {
      "\\.(css|less)$": "<rootDir>/__mocks__/styleMock.js",
      "\\.(gif|ttf|eot|svg)$": "<rootDir>/__mocks__/fileMock.js",

      "^react(.*)$": "<rootDir>/vendor/react-master$1",
      "^config$": "<rootDir>/configs/app-config.js"
    }
  }
}
```

webpack 是一个复杂而灵活的工具，所以你可能需要进行一些必要的调整才能满足您特定应用程序的需求。 幸运的是，对于大多数项目，Jest 应该具有足够的灵活性来处理您的webpack 配置。

> 注意: 对于更复杂的 Webpack 配置，你可能还需要研究一些项目，比如: [babel-plugin-webpack-loaders](https://github.com/istarkov/babel-plugin-webpack-loaders).

## 使用 webpack 2

webpack 2为 ES 模块提供了原生的支持。 但是，Jest 在 Node 中运行，因此需要将ES 模块转换为 CommonJS 模块。 因此，如果您使用的是 webpack 2, 你应该十分乐意将 Babel 配置为仅在“test”环境中将 ES 模块转换为 CommonJS 模块。


```json
// .babelrc
{
  "presets": [["env", {"modules": false}]],

  "env": {
    "test": {
      "plugins": ["transform-es2015-modules-commonjs"]
    }
  }
}
```

> 注意: Jest 会缓存文件来加快测试执行的速度。 如果你更新了.babelrc，而 Jest 仍然无法运行，请尝试使用`--no-cache` 运行Jest。

如果你需要使用动态导入 (`import('some-file.js').then(module => ...)`), 则你需要启用 `dynamic-import-node` 插件.

```json
// .babelrc
{
  "presets": [["env", {"modules": false}]],

  "plugins": ["syntax-dynamic-import"],

  "env": {
    "test": {
      "plugins": ["dynamic-import-node"]
    }
  }
}
```

有关如何在使用 Webpack , React, Redux, 以及 Node 的项目中使用 Jest, 你可以在 [这里](https://github.com/jenniferabowd/jest_react_redux_node_webpack_complex_example) 查看一个示例。
