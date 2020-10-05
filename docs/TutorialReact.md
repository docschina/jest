---
id: tutorial-react
title: Testing React Apps
---

在 Facebook, 我们使用 Jest 来测试 [React](https://reactjs.org/) 应用程序。

## 安装

### 使用 Create React App

如果你是 React 新手，我们建议使用 [Create React App](https://create-react-app.dev/). 它已经包含了 [可用的 Jest](https://create-react-app.dev/docs/running-tests/#docsNav) ! 你只需要添加 `react-test-renderer` 来渲染快照。

运行

```bash
yarn add --dev react-test-renderer
```

### 不使用 Create React App

如果你有一个现有的应用程序，你将需要安装一些包，使一切顺利地协同工作。 我们使用 `babel-jest` 包和 `react` babel 预设在测试环境中转换代码. 另请参阅 [使用 babel](GettingStarted.md#using-babel).

运行

```bash
yarn add --dev jest babel-jest @babel/preset-env @babel/preset-react react-test-renderer
```

你的 `package.json` 应该看起来像这样 (其中 `<current-version>` 是包的实际最新版本号)。 请添加脚本和 jest 配置项：

```json
// package.json
  "dependencies": {
    "react": "<current-version>",
    "react-dom": "<current-version>"
  },
  "devDependencies": {
    "@babel/preset-env": "<current-version>",
    "@babel/preset-react": "<current-version>",
    "babel-jest": "<current-version>",
    "jest": "<current-version>",
    "react-test-renderer": "<current-version>"
  },
  "scripts": {
    "test": "jest"
  }
```

```js
// babel.config.js
module.exports = {
  presets: ['@babel/preset-env', '@babel/preset-react'],
};
```

**准备工作已经完成!**

### 快照测试

让我们来为一个渲染超链接的 Link 组件创建 [快照测试](SnapshotTesting.md) ：

```javascript
// Link.react.js
import React from 'react';

const STATUS = {
  HOVERED: 'hovered',
  NORMAL: 'normal',
};

export default class Link extends React.Component {
  constructor(props) {
    super(props);

    this._onMouseEnter = this._onMouseEnter.bind(this);
    this._onMouseLeave = this._onMouseLeave.bind(this);

    this.state = {
      class: STATUS.NORMAL,
    };
  }

  _onMouseEnter() {
    this.setState({class: STATUS.HOVERED});
  }

  _onMouseLeave() {
    this.setState({class: STATUS.NORMAL});
  }

  render() {
    return (
      <a
        className={this.state.class}
        href={this.props.page || '#'}
        onMouseEnter={this._onMouseEnter}
        onMouseLeave={this._onMouseLeave}
      >
        {this.props.children}
      </a>
    );
  }
}
```

现在，使用 React 的 test renderer 和 Jest 的快照特性来和组件交互，获得渲染结果和生成快照文件：

```javascript
// Link.react.test.js
import React from 'react';
import Link from '../Link.react';
import renderer from 'react-test-renderer';

test('Link changes the class when hovered', () => {
  const component = renderer.create(
    <Link page="http://www.facebook.com">Facebook</Link>,
  );
  let tree = component.toJSON();
  expect(tree).toMatchSnapshot();

  // manually trigger the callback
  tree.props.onMouseEnter();
  // re-rendering
  tree = component.toJSON();
  expect(tree).toMatchSnapshot();

  // manually trigger the callback
  tree.props.onMouseLeave();
  // re-rendering
  tree = component.toJSON();
  expect(tree).toMatchSnapshot();
});
```

当你运行 `yarn test` 或者 `jest`, 将产生一个跟下面类似的文件：

```javascript
// __tests__/__snapshots__/Link.react.test.js.snap
exports[`Link changes the class when hovered 1`] = `
<a
  className="normal"
  href="http://www.facebook.com"
  onMouseEnter={[Function]}
  onMouseLeave={[Function]}>
  Facebook
</a>
`;

exports[`Link changes the class when hovered 2`] = `
<a
  className="hovered"
  href="http://www.facebook.com"
  onMouseEnter={[Function]}
  onMouseLeave={[Function]}>
  Facebook
</a>
`;

exports[`Link changes the class when hovered 3`] = `
<a
  className="normal"
  href="http://www.facebook.com"
  onMouseEnter={[Function]}
  onMouseLeave={[Function]}>
  Facebook
</a>
`;
```

下次你运行测试时，渲染的结果将会和之前创建的快照进行比较。 快照应该随代码更改一起提交。当快照测试失败时，您需要检查它是有意的还是无意的。如果需要更改，可以使用 `Jest-u` 调用 Jest 来覆盖现有快照。

这个例子的代码可以在 [examples/snapshot](https://github.com/facebook/jest/tree/master/examples/snapshot) 中找到.

#### 使用 Mocks, Enzyme 和 React 16 进行快照测试

使用 Enzyme 和 React 16+ 时，快照测试需要注意一些事项， 如果你使用以下方式模拟一个模块:

```js
jest.mock('../SomeDirectory/SomeComponent', () => 'SomeComponent');
```

然后你将在控制台看到如下警告:

```bash
Warning: <SomeComponent /> is using uppercase HTML. Always use lowercase HTML tags in React.

# 或者:
Warning: The tag <SomeComponent> is unrecognized in this browser. If you meant to render a React component, start its name with an uppercase letter.
```

React 16 触发这些警告是因为它检查元素类型的方式，而模拟的模块没有通过这些检查。你可选择的做法是：

1、  作为文本呈现。 这种方式在快照中你将看不到传递给 mock 组件的属性， 但是它很简单直接：
    ```js
    jest.mock('./SomeComponent', () => () => 'SomeComponent');
    ```
2、  作为一个自定义元素呈现。 自定义元素构成的 DOM 没有检查任何内容也不应该会触发警告。 它们是小写的， 并且在名称中带有破折号。
    ```js
    jest.mock('./Widget', () => () => <mock-widget />);
    ```
3、  使用 `react-test-renderer`。 test renderer 不关心元素类型并且可以接受例如 `SomeComponent` 的写法。 你可以使用 test renderer 检查快照， 也可以使用 Enzyme 分别检查组件行为。
4、  禁用警告（需要在 jest 的设置文件中完成)：
    ```js
    jest.mock('fbjs/lib/warning', () => require('fbjs/lib/emptyFunction'));
    ```
    这通常不是一个正确的选择， 因为有用的警告可能会丢失。 然而， 在某些情况下， 例如在测试 react-native's 组件时， 我们将 react-native 标签渲染到 DOM 中会出现许多不相关的警告。 另一个做法是重写 console.warn 方法来抑制特定的警告。

### DOM 测试

如果你想断言和操作渲染的组件， 你可以使用 [react-testing-library](https://github.com/kentcdodds/react-testing-library)， [Enzyme](http://airbnb.io/enzyme/), 或者 React 的 [TestUtils](https://reactjs.org/docs/test-utils.html)。 下面是两个使用 react-testing-library 和 Enzyme 的例子。

#### react-testing-library

你必须运行 `yarn add --dev @testing-library/react` 命令来使用 react-testing-library.

让我们实现一个在两个标签之间切换的复选框：

```javascript
// CheckboxWithLabel.js

import React from 'react';

export default class CheckboxWithLabel extends React.Component {
  constructor(props) {
    super(props);
    this.state = {isChecked: false};

    // 手动绑定 this， 因为 React 组件不会自动绑定
    // https://reactjs.org/blog/2015/01/27/react-v0.13.0-beta-1.html#autobinding
    this.onChange = this.onChange.bind(this);
  }

  onChange() {
    this.setState({isChecked: !this.state.isChecked});
  }

  render() {
    return (
      <label>
        <input
          type="checkbox"
          checked={this.state.isChecked}
          onChange={this.onChange}
        />
        {this.state.isChecked ? this.props.labelOn : this.props.labelOff}
      </label>
    );
  }
}
```

```javascript
// __tests__/CheckboxWithLabel-test.js
import React from 'react';
import {cleanup, fireEvent, render} from '@testing-library/react';
import CheckboxWithLabel from '../CheckboxWithLabel';

// 注意： 在 @testing-library/react@9.0.0 或更高版本中， afterEach(cleanup) 会自动执行
// 测试完成后卸载并清理 DOM。
afterEach(cleanup);

it('CheckboxWithLabel changes the text after click', () => {
  const {queryByLabelText, getByLabelText} = render(
    <CheckboxWithLabel labelOn="On" labelOff="Off" />,
  );

  expect(queryByLabelText(/off/i)).toBeTruthy();

  fireEvent.click(getByLabelText(/off/i));

  expect(queryByLabelText(/on/i)).toBeTruthy();
});
```

这个例子的代码可以在 [examples/react-testing-library](https://github.com/facebook/jest/tree/master/examples/react-testing-library) 中找到。

#### Enzyme

你必须执行 `yarn add --dev enzyme` 命令来使用 Enzyme。 如果你使用的 React 版本在 15.5.0 以下， 你还需要安装 `react-addons-test-utils`.

让我们使用 Enzyme 重写上面的测试而不是使用 react-testing-library。 在这个例子中我们使用 Enzyme 的 [shallow renderer](http://airbnb.io/enzyme/docs/api/shallow.html) 。

```javascript
// __tests__/CheckboxWithLabel-test.js

import React from 'react';
import {shallow} from 'enzyme';
import CheckboxWithLabel from '../CheckboxWithLabel';

test('CheckboxWithLabel changes the text after click', () => {
  // 渲染一个带标签的复选框
  const checkbox = shallow(<CheckboxWithLabel labelOn="On" labelOff="Off" />);

  expect(checkbox.text()).toEqual('Off');

  checkbox.find('input').simulate('change');

  expect(checkbox.text()).toEqual('On');
});
```

这个例子的代码可以在 [examples/enzyme](https://github.com/facebook/jest/tree/master/examples/enzyme) 中找到。

### 自定义转译器

如果你需要更多的高级功能， 你可以建立自己的转译器而不是使用 babel-jest， 下面是一个使用 babel 的例子：

```javascript
// custom-transformer.js
'use strict';

const {transform} = require('@babel/core');
const jestPreset = require('babel-preset-jest');

module.exports = {
  process(src, filename) {
    const result = transform(src, {
      filename,
      presets: [jestPreset],
    });

    return result ? result.code : src;
  },
};
```

别忘记安装 `@babel/core` 和 `babel-preset-jest` 包来使这个例子正常工作。

要想使用 Jest 来实现这一点，你需要更新你的 Jest 配置文件： `"transform": {"\\.js$": "path/to/custom-transformer.js"}`.

如果您想构建一个支持 babel 的转换器，您也可以使用 babel jest 来编写一个，并传入自定义配置项：

```javascript
const babelJest = require('babel-jest');

module.exports = babelJest.createTransformer({
  presets: ['my-custom-preset'],
});
```
