# Setup and Teardown

写测试的时候你经常需要在运行测试前做一些准备工作，和在运行测试后进行一些整理工作。 Jest 提供辅助函数来处理这个问题。

## 为多次测试重复设置

如果你有一些要为多次测试重复设置的工作，你可以使用 `beforeEach` 和 `afterEach`。

例如，我们考虑一些与城市信息数据库进行交互的测试。 你必须在每个测试之前调用方法 `initializeCityDatabase()` ，同时必须在每个测试后，调用方法 `clearCityDatabase()`。 你可以这样做：

```js
beforeEach(() => {
  initializeCityDatabase();
});

afterEach(() => {
  clearCityDatabase();
});

test('city database has Vienna', () => {
  expect(isCity('Vienna')).toBeTruthy();
});

test('city database has San Juan', () => {
  expect(isCity('San Juan')).toBeTruthy();
});
```

`beforeEach` 和 `afterEach` 能够通过与[ 异步代码测试](https://jestjs.io/docs/zh-Hans/asynchronous) 相同的方式处理异步代码 — — 他们可以采取 `done` 参数或返回一个 promise。 例如，如果 `initializeCityDatabase()` 返回解决数据库初始化时的 promise ，我们会想返回这一 promise︰

```js
beforeEach(() => {
  return initializeCityDatabase();
});
```

`beforeEach` 会根据test测试用例的数量遍历循环，如果只想执行一次的话，可以用`beforeAll` 和 `afterAll` 处理这种情况