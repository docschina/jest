---
id: mock-functions
title: 模拟函数
---

模拟函数能够使你通过擦除函数的实际实现、捕获对函数的调用 ( 以及在这些调用中传递的参数) 、在使用`new`实例化时捕获构造函数的实例、允许测试时配置返回值等方法，来测试代码之间的关联。

有两种使用模拟函数的方法: 一种方法是在测试代码中创建一个模拟函数, 另一种方法是编写一个[`手工模拟`](ManualMocks.md)函数来覆盖模块依赖

## 使用模拟函数

假设我们要测试函数`forEach`的内部实现，它为所传入数组的每个元素调用一次回调函数

```javascript
function forEach(items, callback) {
  for (let index = 0; index < items.length; index++) {
    callback(items[index]);
  }
}
```

为了测试这个函数，我们可以使用模拟函数，并且监测模拟函数的状态，以保证回调函数按所期望的形式被调用。

```javascript
const mockCallback = jest.fn(x => 42 + x);
forEach([0, 1], mockCallback);

// 此处，模拟函数被调用两次
expect(mockCallback.mock.calls.length).toBe(2);

// 第一个回调函数的第一个参数为 0
expect(mockCallback.mock.calls[0][0]).toBe(0);

// 第二个回调函数的第一个参数为 1
expect(mockCallback.mock.calls[1][0]).toBe(1);

// 第一个回调函数的返回值为 42
expect(mockCallback.mock.results[0].value).toBe(42);
```

## `.mock` 属性

所有的模拟函数都具有特定的`.mock`属性，该属性包含了函数如何被调用，以及函数的返回值等信息。`.mock`属性也对每个调用的`this`指向进行追踪，因此我们也能够监测`this`：

```javascript
const myMock = jest.fn();

const a = new myMock();
const b = {};
const bound = myMock.bind(b);
bound();

console.log(myMock.mock.instances);
// > [ <a>, <b> ]
```

这些模拟函数的变量属性在测试中十分有用，其能够判断这些函数如何被调用，实例化，以及返回值：

```javascript
// 该函数实际被调用一次
expect(someMockFunction.mock.calls.length).toBe(1);

// 该函数第一次被调用时的第一个参数为 'first arg'
expect(someMockFunction.mock.calls[0][0]).toBe('first arg');

// 该函数第一次被调用时第二个参数为 'second arg'
expect(someMockFunction.mock.calls[0][1]).toBe('second arg');

// 该函数第一次被调用时的返回值为 'return value'
expect(someMockFunction.mock.results[0].value).toBe('return value');

// 该函数实际被实例化两次
expect(someMockFunction.mock.instances.length).toBe(2);

// 该函数的首个实例化对象的一个属性`name`的值被设置为'test'
expect(someMockFunction.mock.instances[0].name).toEqual('test');
```

## 模拟函数的返回值

模拟函数也能被用于将测试值注入你的代码：

```javascript
const myMock = jest.fn();
console.log(myMock());
// > undefined

myMock.mockReturnValueOnce(10).mockReturnValueOnce('x').mockReturnValue(true);

console.log(myMock(), myMock(), myMock(), myMock());
// > 10, 'x', true, true
```

模拟函数在函数连续传递风格类的代码中也十分有效，使用这种风格的代码能够避免采用复杂的中间操作来重建其所代表的实际组件的行为，从而有利于在使用它们之前将值直接注入到测试之中。

```javascript
const filterTestFn = jest.fn();

// 令模拟函数的首次调用时的返回值为`true`，
// 第二次调用时为`false`
filterTestFn.mockReturnValueOnce(true).mockReturnValueOnce(false);

const result = [11, 12].filter(num => filterTestFn(num));

console.log(result);
// > [11]
console.log(filterTestFn.mock.calls);
// > [ [11], [12] ]
```

大多数现实世界例子中，实际是在依赖的组件上配一个模拟函数并配置它，但手法是相同的。 在这些情况下，尽量避免在非真正想要进行测试的任何函数内实现逻辑

## 模拟模块

假设有一个从API获取用户信息的类，该类使用[axios](https://github.com/axios/axios)来调用API，然后返回包含所有用户的`data`属性：

```js
// users.js
import axios from 'axios';

class Users {
  static all() {
    return axios.get('/users.json').then(resp => resp.data);
  }
}

export default Users;
```

现在为了测试上述方法而不实际调用API（其会导致测试缓慢且脆弱），我们可以使用`jest.mock(...)`函数来自动化模拟`axios`模块

一旦对模块进行了模拟，我们就可以为`.get`提供一个`mockResolvedValue`方法，它能够返回我们希望测试针对之断言的假数据。实际上，我们希望axios.get（'/ users.json'）具有一个假的响应。

```js
// users.test.js
import axios from 'axios';
import Users from './users';

jest.mock('axios');

test('should fetch users', () => {
  const users = [{name: 'Bob'}];
  const resp = {data: users};
  axios.get.mockResolvedValue(resp);

  // or you could use the following depending on your use case:
  // axios.get.mockImplementation(() => Promise.resolve(resp))

  return Users.all().then(data => expect(data).toEqual(users));
});
```

## 模拟的实现

不过，在某些情况下，超越指定返回值的功能并完全替换模拟函数的实现是有用的。 这可以通过对模拟函数使用`jest.fn`或`mockImplementationOnce`方法来完成。

```javascript
const myMockFn = jest.fn(cb => cb(null, true));

myMockFn((err, val) => console.log(val));
// > true
```

当需要定义从另一个模块创建的模拟函数的默认实现时，`mockImplementation`方法很有用：

```js
// foo.js
module.exports = function () {
  // some implementation;
};

// test.js
jest.mock('../foo'); // this happens automatically with automocking
const foo = require('../foo');

// foo is a mock function
foo.mockImplementation(() => 42);
foo();
// > 42
```

当你需要重新创建模拟函数的一个复杂行为以使多个函数调用产生不同的结果时，请使用`mockImplementationOnce`方法：

```javascript
const myMockFn = jest
  .fn()
  .mockImplementationOnce(cb => cb(null, true))
  .mockImplementationOnce(cb => cb(null, false));

myMockFn((err, val) => console.log(val));
// > true

myMockFn((err, val) => console.log(val));
// > false
```

当模拟函数用尽了用`mockImplementationOnce`定义的实现时，它将执行用`jest.fn`设置的默认实现（如果已定义）：

```javascript
const myMockFn = jest
  .fn(() => 'default')
  .mockImplementationOnce(() => 'first call')
  .mockImplementationOnce(() => 'second call');

console.log(myMockFn(), myMockFn(), myMockFn(), myMockFn());
// > 'first call', 'second call', 'default', 'default'
```

针对一些我们通常使用链式方法（因此总是需要返回`this`）的情况，我们提供了一个API语法糖————`.mockReturnThis（）`函数以简化这类情况，该函数也位于所有模拟中：

```javascript
const myObj = {
  myMethod: jest.fn().mockReturnThis(),
};

// is the same as

const otherObj = {
  myMethod: jest.fn(function () {
    return this;
  }),
};
```

## Mock名称

您可以选择为模拟函数提供一个名称，该名称将在测试错误输出中显示，而不是显示`jest.fn（)`。 如果您希望能够快速识别在测试输出中报告错误的模拟功能，请使用此功能。

```javascript
const myMockFn = jest
  .fn()
  .mockReturnValue('default')
  .mockImplementation(scalar => 42 + scalar)
  .mockName('add42');
```

## 自定义匹配器

最后，为了减少对如何调用模拟函数的要求，我们为您添加了一些自定义匹配器函数：

```javascript
// 模拟函数至少被调用一次
expect(mockFunc).toHaveBeenCalled();

// 模拟函数至少被调用一次且包含特定的参数
expect(mockFunc).toHaveBeenCalledWith(arg1, arg2);

// 模拟函数最后一次被调用时具有特定的参数
expect(mockFunc).toHaveBeenLastCalledWith(arg1, arg2);

// 所有的调用和模拟的名称被作为快照写入
expect(mockFunc).toMatchSnapshot();
```

这些匹配器是检查`.mock`属性的常见形式的语法糖。 如果这更符合您的口味，或者您需要执行更具体的操作，则始终可以自己手动执行此操作：

```javascript
// The mock function was called at least once
expect(mockFunc.mock.calls.length).toBeGreaterThan(0);

// The mock function was called at least once with the specified args
expect(mockFunc.mock.calls).toContainEqual([arg1, arg2]);

// The last call to the mock function was called with the specified args
expect(mockFunc.mock.calls[mockFunc.mock.calls.length - 1]).toEqual([
  arg1,
  arg2,
]);

// The first arg of the last call to the mock function was `42`
// (note that there is no sugar helper for this specific of an assertion)
expect(mockFunc.mock.calls[mockFunc.mock.calls.length - 1][0]).toBe(42);

// A snapshot will check that a mock was invoked the same number of times,
// in the same order, with the same arguments. It will also assert on the name.
expect(mockFunc.mock.calls).toEqual([[arg1, arg2]]);
expect(mockFunc.getMockName()).toBe('a mock name');
```

有关匹配器的完整列表，请查看[参考文档](ExpectAPI.md).
