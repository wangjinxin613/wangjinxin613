# React的context

Context 意为上下文，提供了一个无需为每层组件手动添加 props，就能在组件树间进行数据传递的方法。

具体详见：[context官方API](https://reactjs.bootcss.com/docs/context.html)

- `React.createContext`
- `Context.Provider`
- `Class.contextType`
- `Context.Consumer`
- `Context.displayName`

### 创建Context  `React.createContext(defaultValue)`


```react
import React from "react";
export const AppContext = React.createContext({
  age: 18
});

```

注意：`React.createContext(defaultValue)` 只有当Provider不存在时，defaultValue才会生效

```react
import "./styles.css";
import React, { useState } from "react";
import Header from "./components/header";
import Content from "./components/content";
import TextContext from "./components/testContext";
import { AppContext } from "./common/context";

export default function App() {
  return (
    <>
      <AppContext.Provider
        value={{
          username: "小王",
          age: 1
        }}
      >
        <div>
          <Header></Header>
          <Content></Content>
        </div>
      </AppContext.Provider>
      <TextContext></TextContext> <!-- 这个组件的defaultValue才会生效 -->
    </>
  );
}


```

### `Context.Provider`

```react
<AppContext.Provider value={/* 某个值 */}>
```

每个 Context 对象都会返回一个 Provider React 组件，它允许消费组件订阅 context 的变化。

Provider 接收一个 `value` 属性，传递给消费组件。一个 Provider 可以和多个消费组件有对应关系。多个 Provider 也可以嵌套使用，里层的会覆盖外层的数据。

当 Provider 的 `value` 值发生变化时，它内部的所有消费组件都会重新渲染。Provider 及其内部 consumer 组件都不受制于 `shouldComponentUpdate` 函数，因此当 consumer 组件在其祖先组件退出更新的情况下也能更新。

通过新旧值检测来确定变化，使用了与 [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is#Description) 相同的算法。

### `Class.contextType`

只有在类组件中指定了contextType的值是哪个Context，才可以在类中使用`this.context`

### `Context.Consumer`

```react
<AppContext.Consumer>
  {(value) => {
    return value.sex;
  }}
</AppContext.Consumer>
```

订阅context的变更

### `Context.displayName`

context 对象接受一个名为 `displayName` 的 property，类型为字符串。React DevTools 使用该字符串来确定 context 要显示的内容。

示例，下述组件在 DevTools 中将显示为 MyDisplayName：

```
const MyContext = React.createContext(/* some value */);
MyContext.displayName = 'MyDisplayName';
<MyContext.Provider> // "MyDisplayName.Provider" 在 DevTools 中
<MyContext.Consumer> // "MyDisplayName.Consumer" 在 DevTools 中
```

没啥用