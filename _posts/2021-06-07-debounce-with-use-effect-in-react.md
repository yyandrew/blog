---
layout: post
title: 使用useEffect 钩子实现防抖效果
date: 2021-06-07 17:56 +0800
---

### 什么是防抖
防抖又称为函数防抖，它是指对于在事件被触发n秒后再执行的回调，如果在这n秒内又重新被触发，则重新开始计时，是常见的优化方法。

### 为什么要防抖

比如有一个实时搜索功能，用户可以根据名字或者标签搜索商品。搜索是当用户打字时立即发生的。如果打开浏览器的开发者工具，你会看到下面的网络请求列表。

![undebounced-requests.png](/images/undebounced-requests.png)

一般请求列表只有最后一个请求返回的结果才是用户想要的。那么问题就出现了，之前的请求都是多余的，它们会浪费服务器宝贵的资源。为了阻止无意义的请求，我们必须对请求进行过滤，防抖节流就是这么来的。

使用防抖之后的网络请求列表：

![debounced-requests.png](/images/debounced-requests.png)

### 如何解决这个问题
解决方法是将请求延时一小段时间，比如0.5秒钟。如果在这0.5秒之内有另外一个请求发生，则清除之前的请求，同时将新的请求延时0.5秒钟。通过这个方式我们能保证0.5秒之内只有一个请求发出。这样就实现了节流防抖。

### 代码示例
创建一个自定义的钩子 useDebouncedSearch
``` javascript
import axios from "axios";
import { useEffect, useState } from "react";
import { API_URL } from "./constant";

const useDebouceSearch = (terms) => {
  const [data, setData] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState();
  useEffect(() => {
    setError(null);
    if (terms) {
      const newTimer = setTimeout(() => {
        (async () => {
          try {
            setLoading(true);
            const res = await axios.get(`${API_URL}/items`, {
              params: { label: terms }
            });
            setData(res.data);
          } catch (error) {
            setError(error);
          } finally {
            setLoading(false);
          }
        })();
      }, 500);
      return () => clearTimeout(newTimer);
  }, [terms]);

  return { data, loading, error };
};
            
export default useDebouceSearch;
```

上面示例代码中关键部分为

1. 使用`setTimeout`创建了一个延时任务，在0.5秒钟之后执行；
2. `useEffect`钩子返回一个函数 `() => clearTimeout(newTimer);`如果 `useEffect` 返回一个函数，刚这个函数会在下次 `useEffect` 之前执行。

完整示例代码: [https://codesandbox.io/s/debounce-with-use-effect-bbvf5](https://codesandbox.io/s/debounce-with-use-effect-bbvf5)
