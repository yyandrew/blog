---
layout: post
title: 为多个组件创建共享函数的一种方法
categories: "React"
date: 2020-09-30 11:52 +0800
---
项目中一定会出现一些公共的函数或者方法，为了遵循[DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)原则。我们应该把这些方法放在哪里呢？下面说说我的在React项目中做法。

在`src`文件夹新建一个`unitls`文件夹，并在这个文件夹创建一个`index.ts`(TypeScript)文件。内容如下：

```typescript
// src/utils/index.ts
import i18n from 'i18next';
import '../i18n';

export const ecommercePrice = (price: number, code: string): string => {
  const currencySymbol = i18n.t(`CurrencyCode.${code.toLowerCase()}`);
  return `${price} ${currencySymbol}`;
};
```

接下来在不同的组件引入这个方法就可以了，比如：

```typescript 
// src/order/EcommercePriceDemo.tsx
import { ecommercePrice } from '../../utils';
const EcommercePriceDemo = (props: IProps): React.ReactElement => {
  return (
  	<div>{ecommercePrice(10, 'EUR')}
  )
}
```



