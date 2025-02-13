---
title: 24-october-10th
tags:
  - date
categories:
  - 日常
sticky: 2
swiper_index: 2
abbrlink: 4a6f3763
date: 2024-10-12 16:32:47
updated: 2025-02-13 18:25:00
---

### 任务

```js

任务分为同步异步任务，

同步先执行，

而后执行异步任务，

异步当中微任务先执行，

每一个宏任务执行完，会再次查看微任务队列，有的话执行微任务队列。

await 语句后面的代码会加入微队列。

注意：await 后面的代码如果是一个宏任务，

那么 await 后面要加入微队列的代码会在 await 后的宏任务执行完之后再执行，这是 await 的特性，会暂停函数中代码的执行，

async await 修饰的函数 ，会等待 await 后面一句的语句执行完，再执行 await 语句下一句的微队列代码，即使 await 后的语句是异

步代码，也会等待这个语句中的异步代码完全执行完。

要注意宏队列、微队列任务在执行过程中 不断添加宏任务、微任务到队列这些任务的执行顺序，执行顺序通常跟添加的顺序有关。

```

### vue hook

```jsx

import { ref } from 'vue';

export const useBoolean = (initValue: boolean = false) => {
  const value = ref(initValue);

  const on = () => {
    value.value = true;
  };
  const off = () => {
    value.value = false;
  };
  const toggle = () => {
    value.value = !value.value;
  };

  return [value, { toggle, on, off }] as const;
};

```

### 比当前背景颜色更深的颜色

```tex
需要两层

一层为当前的颜色

一层为黑色

黑色的这一层是包裹层

这样就能有比当前背景更深一些的颜色

网易云、汽水音乐的背景 ，

也是跟这差不多的实现

```

<img src='https://s3.bmp.ovh/imgs/2024/10/30/5ccc78689055219f.png'></img>

<img src='https://s3.bmp.ovh/imgs/2024/10/30/c2dd0a005c6ec80b.png'></img>
