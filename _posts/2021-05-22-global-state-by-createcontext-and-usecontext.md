---
layout: post
title: 使用Context实现全局状态管理
date: 2021-05-22 08:05 +0800
---

组件之间的状态传递一般是从父向下传给子。如果想要跨层传递状态，需要将状态传递给无关的组件，这样效率太低。

全局状态管理可以高效的实现跨层传递状态。

说到全局状态管理，一般都会选择使用 Redux 插件来实现，但是对于小项目 Redux 有点大材小用。我们也可以使用 React 提供的 Context 及 useReducer hook 实现全局状态管理。

#### 使用 useReducer

useReducer 是 React 提供的一个 hook 。它适用管理逻辑比较复杂且包含多个子值的 state，或者下一个 state 依赖之前的 state 的 state 这种复杂的 state。

useReducer 的工作原理与 Redux 类似，如果你理解 Redux，那么你就已经知道它是如何工作的了。

它需要一种特殊的函数(reducer)来修改状态值。如下图所示：

![reducer-of-use-reducer](/images/reducer-of-use-reducer.png)


同时 useReducer 还需要一个 action 对象用于修改新的状态值。下面是一个 action 示例用来修改 state 中的 name 字段。

``` js
{ type: "SET_NAME", name: "George" }
```

useReducer 能够在函数组件里使用，每当 state 有更新时，组件会被更新。

下面通过一个计数器来演示怎么实现一个全局状态管理，这个计数器包含加一、减一两个按钮以及显示数字的元素。效果如下：
![complete-counter-demo-using-use-reducer-and-use-context.png](/images/complete-counter-demo-using-use-reducer-and-use-context.png)

#### src/AppStateContext.tsx
使用 useReducer 创建一个全局的 state 及 dispatch，同时使用 Context.Provider 将 state 及 dispatch 共享给 App 及 App 的子组件。

最后使用 useContext 获取 state 和 dispatch ，通过调用它我们可以在其它函数组件使用 state 及 dispatch。

``` js
import React, { createContext, useContext, useReducer } from "react";
const AppStateContext = createContext({});
const reducers = (state, action) => {
  switch (action.type) {
    case "INC":
      return {
        ...state,
        count: state.count + 1
      };
    case "DEC":
      return {
        ...state,
        count: state.count - 1
      };
    default:
      return state;
  }
};
const initState = { count: 0 };

export const AppStateProvider = ({ children }) => {
  const [state, dispatch] = useReducer(reducers, initState);
  const currentValue = { state, dispatch }
  return (
    <AppStateContext.Provider value={currentValue}>
      {children}
    </AppStateContext.Provider>
  );
};
export const useAppState = () => {
  return useContext(AppStateContext);
};
```

### src/index.jsx
入口文件
```js
import { StrictMode } from "react";
import ReactDOM from "react-dom";

import App from "./App";
import { AppStateProvider } from "./AppStateContext";

const rootElement = document.getElementById("root");
ReactDOM.render(
  <StrictMode>
    <AppStateProvider>
      <App />
    </AppStateProvider>
  </StrictMode>,
  rootElement
);
```

#### src/App.jsx

``` js
import { useAppState } from "./AppStateContext";

export default function App() {
  const { state, dispatch } = useAppState();
  return (
    <div className="App">
      <button type="button" onClick={() => dispatch({ type: "DEC" })}>
        -
      </button>
      <span>{state.count}</span>
      <button type="button" onClick={() => dispatch({ type: "INC" })}>
        +
      </button>
    </div>
  );
}
```

[codesandbox 源代码](https://codesandbox.io/s/use-createcontext-and-usecontext-to-setup-global-state-jfzzf)
