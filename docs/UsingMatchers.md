---
id: using-matchers
title: 匹配器的用法
---

Jest 使用“匹配器”让你以不同的方式测试值。该部分文档仅介绍一些常用的匹配器。若想查找完整的匹配器列表，请参见[`expect` API 文档](ExpectAPI.md)。

## 常见的匹配器

最简单的测试方法就是判断两个值是否完全相等。

```js
test('two plus two is four', () => {
  expect(2 + 2).toBe(4);
});
```

在这段代码中， `expect(2 + 2)` 返回一个 "expectation" 对象。除了让 "expectation" 对象调用匹配器之外，一般不会对它们做更多处理。上述代码中，`.toBe(4)` 就是匹配器。Jest 运行时会跟踪所有的匹配器，并为失败的匹配项打印错误提示.

`toBe` 相当于 `Object.is`，即完全相等(===)。如果需要检查对象的值，请使用 `toEqual` 代替：

```js
test('object assignment', () => {
  const data = {one: 1};
  data['two'] = 2;
  expect(data).toEqual({one: 1, two: 2});
});
```

`toEqual` 会递归检查数组或对象每一项的值.

你还可以进行反向测试：

```js
test('adding positive numbers is not zero', () => {
  for (let a = 1; a < 10; a++) {
    for (let b = 1; b < 10; b++) {
      expect(a + b).not.toBe(0);
    }
  }
});
```

## 测试真值的匹配器

很多时候测试需要区分 `undefined`、 `null` 和 `false`，为了方便区分，Jest 中定义了专门的匹配器：

- `toBeNull` 只匹配 `null`
- `toBeUndefined` 只匹配 `undefined`
- `toBeDefined` 匹配结果与 `toBeUndefined` 相反
- `toBeTruthy` 匹配 `if` 语句期望得到 true 的
- `toBeFalsy` 匹配 `if` 语句期望得到 false 的

例如：

```js
test('null', () => {
  const n = null;
  expect(n).toBeNull();
  expect(n).toBeDefined();
  expect(n).not.toBeUndefined();
  expect(n).not.toBeTruthy();
  expect(n).toBeFalsy();
});

test('zero', () => {
  const z = 0;
  expect(z).not.toBeNull();
  expect(z).toBeDefined();
  expect(z).not.toBeUndefined();
  expect(z).not.toBeTruthy();
  expect(z).toBeFalsy();
});
```

你应该根据你的代码选择最精确的匹配器。

## 有关数值比较的匹配器

比较数字的大小也有相关的匹配器。

```js
test('two plus two', () => {
  const value = 2 + 2;
  expect(value).toBeGreaterThan(3);
  expect(value).toBeGreaterThanOrEqual(3.5);
  expect(value).toBeLessThan(5);
  expect(value).toBeLessThanOrEqual(4.5);

  // toBe 和 toEqual 对数字的测试结果是一样的
  expect(value).toBe(4);
  expect(value).toEqual(4);
});
```

如果你不希望浮点数的测试存在误差，请使用 `toBeCloseTo` 来代替 `toEqual`。

```js
test('adding floating point numbers', () => {
  const value = 0.1 + 0.2;
  //expect(value).toBe(0.3);           有误差。
  expect(value).toBeCloseTo(0.3); // 无误差。
});
```

## 有关字符串比较的匹配器

字符串的测试可以结合正则表达式使用 `toMatch` 匹配器：

```js
test('there is no I in team', () => {
  expect('team').not.toMatch(/I/);
});

test('but there is a "stop" in Christoph', () => {
  expect('Christoph').toMatch(/stop/);
});
```

## 测试数组或可迭代对象的匹配器

测试数组或可迭代对象是否包含特定元素可以使用 `toContain` 匹配器：

```js
const shoppingList = [
  'diapers',
  'kleenex',
  'trash bags',
  'paper towels',
  'beer',
];

test('the shopping list has beer on it', () => {
  expect(shoppingList).toContain('beer');
  expect(new Set(shoppingList)).toContain('beer');
});
```

## 捕捉函数执行异常的匹配器

如果你想测试某个函数被调用时是否会抛出错误，可以使用 `toThrow`匹配器。

```js
function compileAndroidCode() {
  throw new Error('you are using the wrong JDK');
}

test('compiling android goes as expected', () => {
  expect(compileAndroidCode).toThrow();
  expect(compileAndroidCode).toThrow(Error);

  // 你还可以自定义错误信息提示或者使用正则表达式。
  expect(compileAndroidCode).toThrow('you are using the wrong JDK');
  expect(compileAndroidCode).toThrow(/JDK/);
});
```

## 了解更多

这里只介绍了部分匹配器，完整版请参照 [参考文档](ExpectAPI.md)。

了解了可用的匹配器后，下一步就是检查 Jest 如何让你 [测试异步代码](TestingAsyncCode.md)。
