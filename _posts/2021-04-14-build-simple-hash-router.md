---
layout: post
title: 使用hashchange事件实现一个简单的单页应用路由
date: 2021-04-14 15:35 +0800
cagetories: '读书笔记'
---

1. 在`Router.js`里创建一个Router组件，它的作用是监控URL的hash更新，更新state值，进而更新页面内容;
2. 在首页`App.js`里渲染Router组件;
3. 在`App.js`里定义`mapping`对象存储键是URL的hash和其对应的值是页面需要展示的内容。

* 项目结构

```
router-demo
├── package.json
├── public
│   └── index.html
└── src
    ├── App.js
    ├── Router.js
    └── index.js
```

* App.js



```javascript
import Router from "./Router";

export default function App() {
  // mapping对象，包含三个键。分别是：#projects, #profie, *
  const mapping = {
    "#project": (
      <div>
        {" "}
        Project <a href="/#">Back to Dashboard</a>{" "}
      </div>
    ),
    "#profile": (
      <div>
        {" "}
        Profile <a href="/#">Back to Dashboard</a>{" "}
      </div>
    ),
    "*": (
      <div>
        Dashboard
        <br />
        <a href="#project">Project</a>
        <br />
        <a href="#profile">Profile</a>
        <br />
      </div>
    )
  };
  return (
    <div className="App">
      <Router mapping={mapping} />
    </div>
  );
}
```

* Router.js



``` javascript
import React from "react";
export default class Router extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      hash: window.location.hash // 获取URL里的hash值
    };
  }
  updateHash = () => {
    this.setState({ hash: window.location.hash }); // 更新state状态值
  };
  componentDidMount() {
    window.addEventListener("hashchange", this.updateHash, false); // 虚拟组件渲染成DOM之后绑定`hashchange`事件来监控URL的变化
  }
  componentWillUnmount() {
    window.removeEventListener("hashchange", this.updateHash, false); // DOM销毁之后应该删除事件，防止潜在的内存泄漏
  }
  render() {
    if (this.props.mapping[this.state.hash]) {
      return this.props.mapping[this.state.hash]; // 更新页面内容
    } else {
      return this.props.mapping["*"]; // 访问不存在的URL hash时，默认使用dashboard页面
    }
  }
}
```

项目地址：[https://codesandbox.io/s/lingering-fast-fidge](https://codesandbox.io/s/lingering-fast-fidge)