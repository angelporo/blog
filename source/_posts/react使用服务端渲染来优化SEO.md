---
layout: post
title: react使用服务器渲染来优化SEO
date: 2018-02-26 17:46:10
tags:
    - react
    - seo
    - react服务器渲染
---

seo我认为这个就是搜索引擎没有吧这个东西做好, 然后就把这个抛给了开发者(百度), 看看google, 在2015年就把这个事情给解决了, 不光分析你的`html`代码还分析你的`css`和`javascript`脚本
不过没有办法谁让我们出门不用穿防弹衣呢!!!

动态页面想要做好seo优化, 在网上看了看常用的有两种
- 使用`PhantomJS`爬虫来解决(改变html内容结构)
- 使用服务端渲染(使用react的`renderToString`使用服务器来输入renderToString之后的html内容)

# react, redux, react-router配合使用实例
react中提供了两个方法`renderToString` 和 `renderToStaticMarkup` 用来将组件（Virtual DOM）输出成 HTML 字符串，这是 React 服务器端渲染的基础，它移除了服务器端对于浏览器环境的依赖，所以让服务器端渲染变成了一件有吸引力的事情。

想要达到服务器渲染除了解决浏览器环境的依赖, 还要解决两个问题
- 前后端可以共享代码
- 前后端路由可以统一处理

这里我们使用`react-router`来管理路由, 使用`redux`来管理stroe

这里不介绍`redux`的使用, 大家可以[redux中文](http://cn.redux.js.org/index.html)来学习redux

## react-router
[`react-router`](https://github.com/ReactTraining/react-router)是通过一种声明式的方式匹配不同路由来分配不同组建内容的, 通过`props`来变更路由然后出发`re-render`

假设有一个很简单的应用，只有两个页面，一个列表页 `/list` 和一个详情页 `/item/:id`，点击列表上的条目进入详情页。

可以这样定义路由，`./routes.js`

```javascript
import React from 'react';
import { Route } from 'react-router';
import { List, Item } from './components';

// 无状态（stateless）组件，一个简单的容器，react-router 会根据 route
// 规则匹配到的组件作为 `props.children` 传入
const Container = (props) => {
  return (
    <div>{props.children}</div>
  );
};

// 配置route 规则：
// - `/list` 显示 `List` 组件
// - `/item/:id` 显示 `Item` 组件
const routes = (
  <Route path="/" component={Container} >
    <Route path="list" component={List} />
    <Route path="item/:id" component={Item} />
  </Route>
);

export default routes;
```

上面可以看出我们指定了一个列表页 `/list` 和一个详情页 `/item/:id`，点击列表上的条目进入详情页。分别对应组建`List`和`Item`组建
接下来我们就通过这个简单的例子来实现服务器渲染的细节.

## Reducers
redux中store是由reducer产出的, 所以reducer实际上反应了Store的树结构

`./reducers/index.js`

```javascript
import listReducer from './list';
import itemReducer from './item';

export default function rootReducer(state = {}, action) {
  return {
    list: listReducer(state.list, action),
    item: itemReducer(state.item, action)
  };
}
```

`rootReducer`的`state`参数就是一个Store的状态树, 状态书下的内阁字段对应也可以有自己的`reducer`, 这个引入`ListReducer`和`itemReducer`, 可以看到这两个reducer的state参数就是一个整个状态树对应的list和item字段.

具体看`./reducers/list.js`

```javascirpt
const initalState = [];
export default function listReducer (state = initialState, action) {
  switch(action.type) {
  case 'FETCH_LIST_SUCCESS': return [...action.payload];
  default: return state;
  }
}
```

list就是一个包含items的数组, 结构类似[{id: 0, name: 'li'}, {id: 1, name: "wang"}], 通过`actions.type`获取相应的`store`具体值

继续看`./reducers/item.js`

```javascript
const initialState = {};

export default function listReducer (state = initialState, action) {
  switch(action.type) {
  case 'FETCH_ITEM_SUCCESS': return [...action.payload];
  default: return state;
  }
}
```

## Action

对应的想要修改store就必须要有两个action来获取list和item, 触发reducer改变store, 然后返回新的store, 从而改变渲染内容, 

`./actions/index.js`

```javascript
import fetch from 'isomorphic-fetch';

export function fetchList() {
  return (dispatch) => {
    return fetch('/api/list')
        .then(res => res.json())
        .then(json => dispatch({ type: 'FETCH_LIST_SUCCESS', payload: json }));
  }
}

export function fetchItem(id) {
  return (dispatch) => {
    if (!id) return Promise.resolve();
    return fetch(`/api/item/${id}`)
        .then(res => res.json())
        .then(json => dispatch({ type: 'FETCH_ITEM_SUCCESS', payload: json }));
  }
}
```

通过调用`dispatch`出去相应的`action`函数来达到具体操作`store`中的值

[isomorphic-fetch](https://github.com/matthew-andrews/isomorphic-fetch)是一个后端实现`Ajax`的实现, 
这里具体涉及到异步请求, 这里的`action`用到了`thunk`, redux通过`thunk-middewar`来处理异步action, 把函数当做普通的action dispatch就好了, 无非就是在action内部可以拿到dispatch函数来操作store. 通常看成成普通高阶函数就ok.

## Store

这里用独立的`./store.js`, 配置生成的`store`
```javascript
import { createStore } from 'redux';
import rootReducer from './reducers';

// Apply middleware here
// ...

export default function configureStore(initialState) {
  const store = createStore(rootReducer, initialState);
  return store;
}
```

react-redux

接着我们就实现`<List>`, `<Item>`组建,  然后把Redxu和react组件关联起来, 具体细节参见`redux`教程

`./app.js`入口文件


```javascript
import React from 'react';
import { render } from 'react-dom';
import { Router } from 'react-router';
import createBrowserHistory from 'history/lib/createBrowserHistory';
import { Provider } from 'react-redux';
import routes from './routes';
import configureStore from './store';

// `__INITIAL_STATE__` 来自服务器端渲染，下一部分细说
const initialState = window.__INITIAL_STATE__;
const store = configureStore(initialState);
const Root = (props) => {
  return (
    <div>
      <Provider store={store}>
        <Router history={createBrowserHistory()}>
          {routes}
        </Router>
      </Provider>
    </div>
  );
}

render(<Root />, document.getElementById('root'));
```

上面全部部分都是我们的客户端代码.(就是一个简单的react, 配合react-rout和redux的demo)

下面我们的重头戏来了

# Server Redering 服务端渲染

对于上面能看懂的朋友, 下面就相对简单了, , , (难点就在redux的使用上面) 获取数据可以调用action,
在服务器端用一个`match`(react-router提供) 方法将拿到的request url匹配到我们之前定义的routes, 解析成和客户端一致的props对象传递给组件.

`./server.js`

我们使用express来渲染(这里是离不开node的, 别想着离开node来做这件事)

```javascript
import express from 'express';
import React from 'react';
import { renderToString} from 'react-dom/server';
import { Provider } from 'react-redux';
import { RoutingContext, match } from 'react-router';
import routes from './routes';
import configureStore from './store';

const app express();


/**
 * 装载满的页面
 */
function  renderFullPage( html,  initialState) {
    return `
    <!DOCTYPE html>
    <html lang="en">
    <head>
      <meta charset="UTF-8">
    </head>
    <body>
      <div id="root">
        <div>
          ${html}
        </div>
      </div>
      <script>
        window.__INITIAL_STATE__ = ${JSON.stringify(initialState)};
      </script>
      <script src="/static/bundle.js"></script>
    </body>
    </html>
    `;
}

app.use( (req, res ) => {
    match( { routes, location:req.url }, ( err, redirectLocation, renderProps ) => {
        if (err) {
            res.state(500).end(`这里出错了!具体错误:${err}`);
        }else if ( redirectLocation) {
            res.redirect(redirectLocation.pathname + redirectLocation.search);
        }else if (renderprops) {
            const store = configureStore(); // 这里和客户端渲染使用的同一个store
            const state = store.getState();
            Promise.all([
                store.dispatch(fetchList()),
                store.dispatch(fetchItem(renderProps.params.id))
            ])
            .then(() => {
                const html = renderToString(
                <Provider store={store}>
                <RoutingContext {...renderProps} />
                </Provider>
            );
            res.end(renderFullPage(html, store.getState()));
            });
        } else {
            res.status(404).end("Not found");
        }
    })
})

```

上面已经标出了和客户端使用的同一个`Store`数据, 另外注意`renderFullPage`生成的页面html在react组件mount的部分(<div id="root"), 前后端的html结构应该是一致的, 然后要把`store`的状态书写入一个全局变量(`__INITIAL_STATE__`), 这样客户端初始化render的时候能够校验服务器生成的html结构, 并且同步到初始化状态, 然后整个页面被客户端接管.

# 总结

页面内部链接跳转使用`react-router`提供的`<Link>`组建代替`<a>`标签就ok了, 其他交给`react-router`就ok

这里已经全部结束了, 总结一下,  是不是觉得很简单,  很大一部分是说了`redux`的基本用法, 真正用到的只是`./server.js`来渲染,
其中使用express中间件来监听`request`和`result`来使用`react-router`提供 match 方法来拿到当前的客户端路由然后后端通过`renderToString`渲染成字符串返回给客户端.

还有一种优化seo的方式是通过爬虫的方法,  但是我不喜欢那种, 有点作弊的嫌疑,  如果真的被判定作弊, 那么就不是优化的问题了.
