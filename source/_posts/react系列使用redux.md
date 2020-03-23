---
title: react系列使用redux
date: 2017-06-14 09:50:15
tags:
  - redux
  - react
categories: "react"
comments: false
---

## redux
附上[中文文档](http://cn.redux.js.org/index.html)

> Redux 是 JavaScript 状态容器，提供可预测化的状态管理,
redux 出现是时间并不长, 是有flux发展而来

### 设计动机

目前单页面应用越来越多, 而且需要维护的状态也越来越复杂，诸如维护数据更新、UI更新、本地数据存储等这些都是我们在js应用常常需要处理的情景，当然这里多数都会涉及到异步处理。而redux本身就是为了解决这些问题，而是将所有的变化进行统一流程处理，会使我们的程序状态变化清晰可见。redux最终目的就是让状态(state)变化变得可预测。

<!-- more -->

### 查看文档

之前刚开始看redux的时候, 我脑子里面想的就是这东西怎么这样子,  第一次看的时候感觉只明白了counter demo, 希望各位大佬们可以耐心的看文档
好吧, 最终 ! 我看了不下5遍才看的明白, 源码相当精剪, 其中有中间件这个概念, 看过node express的应该对中间件比较了解.

### 使用原则

> Redux里的强硬规则与设计不少，大部份都会与FP(函数式程序开发)、改进原本的Flux架构设计有关。Redux官网文档上的三大基本原则，
> 主要是因为有可能怕初学者不理解Redux中的一些限制或设计，所以先写出来说明，这里面也说明了Redux的设计原理基础是如何，所以强烈建议所有的初
> 学者一定要彻底地理解这三大原则中的意义，多看几遍，对日后的学习会很有帮助。以下分别说明，主要以原文的标题与内容说明，尽可以说明的比较清楚些。

1. Single source of truth

 单一数据来源, 整个应用的state，存储在唯一一个object中，同时也只有一个store用于存储这个object.

2. State is read-only

 状态是只读的, 唯一能改变state的方法, 就是触发action操作, action是用来描述正在发生的事件的一个对象. (action是一个对象, 其中不可或缺的是type属性).

3. Changes are made with pure functions

 在改变state tree 用到action，同时也需要编写对应的reducers才能完成state改变操作。

> 附上官网经典案例 counter

```javascript
import { createStore } from 'redux';

/**
 * 这是一个 reducer，形式为 (state, action) => state 的纯函数。
 * 描述了 action 如何把 state 转变成下一个 state。
 *
 * state 的形式取决于你，可以是基本类型、数组、对象、
 * 甚至是 Immutable.js 生成的数据结构。惟一的要点是
 * 当 state 变化时需要返回全新的对象，而不是修改传入的参数。
 *
 * 下面例子使用 `switch` 语句和字符串来做判断，但你可以写帮助类(helper)
 * 根据不同的约定（如方法映射）来判断，只要适用你的项目即可。
 */
function counter(state = 0, action) {
  switch (action.type) {
  case 'INCREMENT':
    return state + 1;
  case 'DECREMENT':
    return state - 1;
  default:
    return state;
  }
}

// 创建 Redux store 来存放应用的状态。
// API 是 { subscribe, dispatch, getState }。
let store = createStore(counter);

// 可以手动订阅更新，也可以事件绑定到视图层。
store.subscribe(() =>
  console.log(store.getState())
);

// 改变内部 state 惟一方法是 dispatch 一个 action。
// action 可以被序列化，用日记记录和储存下来，后期还可以以回放的方式执行
store.dispatch({ type: 'INCREMENT' });
// 1
store.dispatch({ type: 'INCREMENT' });
// 2
store.dispatch({ type: 'DECREMENT' });
// 1
```

在上面的三原则中，我们看到了store, action, reducer这些词，那就先说说redux是怎么进行应用状态(state)维护管理的呢。

## redux 状态管理的流程

- action是用户触发或程序触发的一个普通对象

- reducers是更具action操作来做出不同的数据响应, 返回一个新的state. (务必有返回state, 返回的state就是store的结果)

- store 最终的值就是有reducers来确定的 . (一个store是一个对象, reducer会改变tore中的某些值)

{% asset_img redux-thing-image1.png redux流程 %}

上图可以简单的看出redux的改变 状态 . action -> reducer -> 新store -> ui更新

下面是具体的实例:

{% asset_img redux-thing-image2.png redux登录具体流程 %}

store用于维护状态的容器，包括了应用的多个状态，比如说用户是否登录、用户信息、用户任务等等。action是一个普通对象，用于指明是哪种操作，这样才能在reducers中进行识别。而众多reducer是负责返回新的state的函数。在实际应用中，你需要将store或store的某个值绑定到界面，这样更新store的时候，该页面可以监听到值的更新，然后进行一些页面更新操作/跳转操作等。

> [原文地址](http://www.jianshu.com/p/2c43860b0532)

### 推荐两个react和redux的调试小工具
    [React Developer Tools Chrome](http://www.cnplugins.com/devtool/react-developer-tools/)
    [Redux Dev Tools Chrome](http://www.cnplugins.com/devtool/redux-devtools/)
