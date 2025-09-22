---
title: react-custom-hook-four
tags:
  - reacthook
categories:
  - react
mathjax: true
abbrlink: 346c1453
date: 2025-04-28 09:22:08
description:
---

### useIsFirstRender

```tex
这个hook 会在首次渲染的时候返回一个为true的布尔值 

为什么需要这个hook？

我们常常在组件首次渲染的时候需要执行一些方法 ，或者是首次指定默认值，（在一些组件中指定默认值失效的情况下会非常有用）
```

```tsx
export const useIsFirstRender = (deps: [] as unknown[], noDeps: [] as unknown[]) => {
  const isFirstRender = useRef(false)
  
  /**
  * 
  * 为什么需要指定依赖项？
  * 
  * 页面首次渲染可能会更新多次 多个状态或props的先后更新会导致 即便是首次加载页面 组件也会多次更新
  *  
  * 这将会导致我们的hook记录失效！ß
  *  
  * 因此 只在我们指定的状态更新才开始计数。
  */
  useEffect(() => {
    isFirstRender.current = false;
  },[...deps])
  
  /**
  * 在有些状态变化的情况下
  *
	* 我们依然希望使用默认的初始值，这个时候你应该指定这些变量，来在他们更新时，返回true来保证使用的仍然是初始值
  */
  useEffect(() => {
    isFirstRender.current = true;
  },[...noDeps])
  
  useEffect(() => {
		isFirstRender.current = true;
    return () => {
      sFirstRender.current = false;
    }
  },[])
  
  return {
    isFirstRender: isFirstRender.current
  }
}
```

### useDepsNoRender

```tex
如你所见

当状态更新时，不执行回调的hook。

为什么需要这个hook?

因为组件的bug，我们在回显时不能回填数据（数据会丢失，即便你赋值前值是全量的），

需要多次更新才能正常回显，

因此在页面首次加载时，通过多次渲染回填数据。

首次渲染即使依赖更新了，我们依然执行回调，来完成数据回显。

而在后续的渲染中（数据已经正常回填，组件中其他状态发生了改变，导致组件渲染此时不会执行回调，

前提是你把导致组件渲染的状态写入了这个hook的依赖）

他是一个数据回填但数据丢失，

你需要在首次加载回显数据，而后续不再执行的一个hook。
```

```tsx
export const useDepsNoRender = (callback: () => void, deps: unknown[], firstRender = true) => {
  const isUpdate = useRef(false)
  const { isFirstRender } = useIsFirstRender(deps)
  
  useEffect(() => {
    isUpdate.current = true
  })
  
  useEffect(() => {
    isUpdate.current = false
  }, [...deps])
  
  useEffect(() => {
    if (firstRender && isFirstRender) {
      isUpdate.current = true
    }
  }, [firstRender, isFirstRender])
  
  useEffect(() => {
    if(isUpdate.current) callback?.()
  })
}
```

### usePerformanceModel

```tex
简化umi useModel 的取值 有类型提示
```

```tsx
import { useModel } from 'umi';
// Parameters 利用提取 useModel 第一个参数，来取到所有的 ModelName
type ModelNames = Parameters<typeof useModel>[0];
// 根据指定的 ModelName 来拿到 useModel 的返回值（对应的 model state）
type ModelState<N extends ModelNames> = ReturnType<typeof useModel<N>>;

export const usePerformanceModel = <N extends ModelNames, K extends keyof ModelState<N>>(
  modelName: N,
  keys: K[]
) => {
  return useModel(modelName, (state) => {
    return keys.reduce((prev, key) => {
      if (prev[key]) return prev;
      prev[key] = state[key];
      return prev;
    }, {} as Pick<ModelState<N>, K>);
  });
};
```

