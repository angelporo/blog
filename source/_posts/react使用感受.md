---
title: react使用感受
date: 2017-06-10 16:23:21
tags:
categories: "blog"
comments: false
---

{% asset_img react.png react 官网 %}

<!-- more -->

## react感受

记得刚开始接触[react](https://facebook.github.io/react/)的时候还是在前年那时候react版本还在零点几,
github上面的star也是在3w左右, 随着时间的变化和react社区的活跃, 现在star已经马上到6.9w了, 可见react的发展之快, 当初还是angular的天下...
现在想起来也挺庆幸能认识上个工作的同事.[送上github传送门](https://github.com/yuffiy)(也是现在的同事)

> 当初自己是在官网上看的资料,  学习一个东西无非就是上官网, 或者看大佬做的视屏, 或者论坛里面看帖子学习
> qq群就算了吧!那里都是装X的, 也不是看东西的地方, 只能是聊起天来比较爽

### webpack打包

这块我认为还是很重要的, react, vue, angular比较起来学习成本高, 很大一部分就在打包上面,
需要掌握写nodejs和 webpack 打包工具, 初学者看起来可能会有点头晕, 不过加以理解, 还是不难的, 网上也有很多[]()webpack的教程, 以及一些优化教程, 英语水平好的话可以直接看官网, 可以少走不少弯路.
顺便插一句:

> 编程写到一定瓶颈了, 一定是英语阅读能力拖后退,  so Study English hard.

webpack打包工具配置好, 这时候才是真正开心编写代码的时候,其中打包还涉及到单页面和多页面的区别, 还有就是全局变量控制错误处理, 单页面需要react-router来通过路由的方式来操作页面的切换, 多页面需要对打包性能优化做一定的处理,  如果学习起来困难可以看看<<慕课网>>里面的webpack视屏, 讲的很清晰

### react state和props的设计

#### state和props的区别

下面的感受是结合自己写 函数式柯里化 悟出来的, 讲的不对的希望自己给予建议

> 每一个State组件都可以拥有自己的state，state与props的区别在于state只存在于组件的内部。state可以用来确定一个元素的view状态。
> 所以在设计的时候state尽可能的来控制view的状态, 就如函数内部使用闭包之后的局部变量 .这样组件与组件之间就不会有太多的交际, 但是组合成高阶组件之后props看起来也很清晰.
> props则是由父组件 或者 实例化 的时候传递进去的可变参数, 就如通过传 输传递到函数内部的全局变量

react刚开始写的时候会觉得component中的state不能在其他component中使用, 如果使用同一个state还需要同能回调函数传递它们两个组件的功能父组件, 因为react数据流设计初衷(单项数据流)只能
父元素向子元素 所以就出现了这种回调地域的写法,  自从使用redux才解决了这个回调地域的困境


## redux
附上[中文文档](http://cn.redux.js.org/index.html)

> Redux 是 JavaScript 状态容器，提供可预测化的状态管理。

> 之前刚开始看redux的时候, 我脑子里面想的就是这东西怎么这样子,  第一次看的时候感觉只明白了counter demo
> 好吧, 最终 ! 我看了不下5遍才看的明白, 源码相当精剪, 其中有中间件这个概念, 看过node express的应该对中间件比较了解

### 附上官网counter案例

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

> 后续 -> react配合redux开发手机端页面感受
