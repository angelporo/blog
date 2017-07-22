---
title: 优雅的设计redux中的store的state树
date: 2017-06-15 15:04:08
tags:
  - redux
  - react
  - react-nativem
categories: "redux"
comments: false
---

## 如何优雅的设计redux的store中的state树

>使用redux几个月的时间, 估计最大的问题就是怎么设计store才是合理的?
>是这个问题估计让我思考了好就才想到问题的关键处.
> state树按页面规划 还是按照数据库中的表划分

<!-- more -->

### 按照页面划分

> 是吧每个页面当成一个业务模块, 实际上这也是非常流行的一种做法, 很多公司就是这样子做的, 可以像下面这样初始化你的store, 适合多页面开发使用.

```javascript
module.exports = function() {
  return {
    common: {
      isFetching: false
    },
　　 profile: {
　　 },
    list: {
    },
　　 edit: {
　　 },
    home: {
    }
  }
}
```
- 优点

	> 模块之间互相独立,  不会互相影响, 每个页面维护自己的reducer, 由于模块中需要展示的东西不多所以数据量不是很大,使用起来很方便, 不太需要其他数据的缓存数据, 比较符合redux的设计(并不是用来做一个前端数据库)

- 缺点

		但这种方法有个致命的缺点，就是store中数据的同步问题和冗余问题。
> 假如一个列表页面的数据依赖于另一个编辑页面的操作，在编辑页面，改变了数据.
>这时候数据库里面的数据已经被改变了，但这时候列表页面和编辑页面的state是独立的，列表页面state节点下的数据并没有改变，退回到列表展示页，数据并不会发生变化，这就有问题了，而且同一份数据出现在多个节点下，也会有数据冗余的问题。

	解决方法:

	1. 每次进入列表页面去数据库刷新store  (如果你做的是一个用户端App显然不会这么干，只可能在第一次进入的时候才会载入数据，第二次应该就不会发请求了，立即从store中拿出缓存数据，否则性能和体验太差，PC端后台倒是无所谓)

	2. 编辑完保存后dispatch一个或多个action，去改变所有依赖到这条数据的页面的state (相对还是比较可靠的办法，前提是依赖的页面很少，如果是一个庞大的系统，每次新增加一个页面，你就需要去修改派发action的代码，太麻烦了，除非建立一个完备的事件系统，所有依赖到这条数据的页面都监听一个事件，只要一编辑，就触发所有的事件，更新自己的state节点)

		具体操作:  当数据源保存成功后`发布`一个全局事件, 在依赖这个数据源的地方`订阅`这个事件, 在订阅的函数里面指定一个保存store的action

	3. 编辑完后除了更新数据库中的数据，在前端这边什么都不做，返回列表页，先取出缓存数据展示，等到页面DidMount之后悄悄发一个请求，不要出loading图，检查一下数据是否被更新过，如果是，拉取最新数据并更新store，如果不是，那就不变

    可以看看[RxJS](https://github.com/Reactive-Extensions/RxJS)

	4. 列表页和编辑页共用一个state数据节点，列表页只存必要显示的字段，点击进入编辑页，根据列表页的记录id读取更详细的信息，编辑保存完成后，更新到这个state节点（这种方法其实采用的就是按数据库表划分的办法，统一数据源，关于统一数据源格式，可以看一看列表页和编辑页共用一个state数据节点，列表页只存必要显示的字段，点击进入编辑页，根据列表页的记录id读取更详细的信息，编辑保存完成后，更新到这个state节点（这种方法其实采用的就是按数据库表划分的办法，统一数据源，关于统一数据源格式，可以看一看[normalizr](https://github.com/paularmstrong/normalizr)

#### 同时使用react组件中的state和redux中store中的state

> 注意：没有人告诉过你，用了redux必须通篇要用store中的state，redux作者也没有这么讲过，比较好的做法是结合react组件自己的state和store中的state一起使用。

### 按照数据库中的表划分,比如想下面这样init 你的store

```javascript
module.exports = function() {
  return {
    common: {
      isFetching: false
    },
　  companies: {
    },
    users: {
    },
    events: {
    }
  }
}
```

> 这种方法可能比较适合单页应用，把store当成数据库使，增删改查都维护这一个store，作为前端数据源，保证前端数据源和后端数据库同步，并且保证了操作的及时反馈，适合对用户体验要求较高的场景，缺点就是前端不像后端，除了model数据，还有业务数据，临时状态数据，不能简单的用数据库的思路去设计store，如果是这样，那么为什么不用前端数据库呢？为啥还要用redux来多此一举，redux本质上是用来管理数据状态的。

	所以, 我认为还是需要按照页面来划分, 但是应该和数据库划分表结合起来

在此之前, 应该分清楚那些数据是放在store中, 如果是零时数据不需要做缓存, 我们应该用react自己的state去维护状态, 而不是放在redux中的store中.

如果该数据需要缓存, 缓存只被一个页面所依赖, 那么就放到依赖页面的state中.

如果该数据需要做缓存,并且不是被一个页面所依赖, 那么我们就把该数据提取出来, 放到一个公共state节点下

redux作者的意思大概就是，对于store中的state树，躺着用，站着用，跪着用，想怎么用怎么用，并没有明确规定必须怎么用，只要自己用的舒服，解决你自己的特殊场景就可以了。

参考:

[http://www.jianshu.com/p/f3911358ebcb](http://www.jianshu.com/p/f3911358ebcb)

[https://www.zhihu.com/question/47995437?sort=created](https://www.zhihu.com/question/47995437?sort=created)

[https://www.zhihu.com/question/50888321](https://www.zhihu.com/question/50888321)
