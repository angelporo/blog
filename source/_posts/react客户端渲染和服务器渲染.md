---
layout: post
title: react客户端渲染和服务器渲染
date: 2018-03-28 09:21:08
tags:
    - react
    - react 服务器渲染
categories: "react"
---

这里这是一个铺垫,因为使用`react redux`的小伙伴应该都有一个困惑,就是大多数些项目的人都需要`create-react-app`来意一步步构建自己的项目,然后需要些很多同样的代码,这也是我遇到的问题,所以打算写一个`create-redux-app`的脚手架,然后提交到`npm`上面,之后就可以开箱即用的脚手架来开发自己的`redux react-router`项目,项目基于`react-start-kit`来写...[这里是github项目地址(待完成)]()


> 本文有点多,是基于晚上很多文章整合

# 为什么要前后端同构？

简单列几点：

- 相对于纯前端渲染来说，更利于 SEO
- 相对于纯前端渲染来说，首屏加载更快
- 提高了复用性，代码更容易维护

TODO: 这里需要配图

# 实现同构需要解决的问题

理想是丰满的，现实是骨感的，在实现前后端同构的路上，我们会不可避免的遇到几个问题：

# 运行环境差异

有些逻辑是只能运行于浏览器端的，比如 `AJAX` 请求，浏览器事件等等，应置于` componentDidMount` 生命周期（含）以后，或者通过 window 关键字进行判断。

# 模块管理差异

我们通过 `Webpack`，可以实现样式、图片等各种资源的模块化管理，但 `Node` 的` require` 可没有 `Webpack` 的魔法特性，我们可以通过` webpack-isomorphic` 工具来解决。


# 前端渲染数据来源

服务端渲染时，`React` 组件的` state` 只存在于服务器内存中，而生命周期方法只会执行到 `componentDidMount` 以前，那么前端如何将组件重新实例化，并将剩余的生命周期方法执行完呢？我们需要把服务端数据同步到前端，前端的 `React` 才能够在浏览器中进行组件的实例化，保证 `Web` 应用正常运行。

# 前后端路由同步

如果你的应用是单页应用（SPA），那么应该确保前后端路由一致，否则可能出现页面跳转可以正常访问，但刷新后却出现异常的情况。

# 服务端渲染

这里不细说,[react使用服务端渲染来优化SEO](http://angely.me/2018/02/26/)说的很明白


# 为服务端实现 require 魔法
上一篇我们提到过，我们可以通过 Webpack 加载器将样式、图片等前端资源作为模块进行引入，但这些资源如果在 Node 中直接引入，是会报错的，因此，我们需要借助 webpack-isomophic 模块来消除前后端 require 函数的差异。

## 前端部分
在 webpack.config.js 中，配置 webpack-isomorphic 插件：

```nodejs
var IsomorphicPlugin = require('webpack-isomorphic/plugin');
// 配置需要同构的文件后缀名
var isomorphicPlugin = new IsomorphicPlugin({
    extensions: ['jpg', 'png', 'gif', 'css']
});
module.exports = {
    // 配置源文件目录
    context: __dirname + '/src',
    // ...
    plugins: [
        //...
        isomorphicPlugin
    ]
};
```

## 服务端部分
```nodejs
var webpackIsomorphic = require('webpack-isomorphic');
// 配置构建后的文件目录
webpackIsomorphic.install(__dirname + '/dist', {
    cache: process.env['NODE_ENV'] !== 'development'
});
```

不得不说 webpack-isomorphic 的使用确实非常的简单方便。

# 前后端数据同步
前面提到，React 在浏览器端，如果 virtual DOM 的渲染结果与服务端一致，则不需要重新渲染 DOM 树，因此前端的数据也应该与服务端保持一致，那么前端如何得到服务端的数据呢？方法也很简单，向浏览器输出 HTML 的时候，同时把数据以 JSON 的格式置于 <script> 标签中，输出至浏览器。


# 服务器

输出页面 HTML 时，同时带上原始数据：


```js
var initialData = {foo: 'bar'};
var viewsDir = path.join(__dirname, './views/dist');
var template = path.join(viewsDir, 'index.tpl');
var reactClass = path.join(viewsDir, 'js/index.js');
var factory = React.createFactory(require(reactClass));
gotpl.renderFile(template, {
    initialData: initialData,
    initialHTML: ReactDOMServer.renderToString(factory(initialData))
}, function (err, html) {
    if (err) {
        res.end(err.stack);
    } else {
        res.end(html)
    }
});
```

# HTML模板

将原始数据置于 <script> 标签中，并赋值给 window.initialData：

```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>React+Webpack 前后端同构</title>
    <link rel="stylesheet" href="index.css">
</head>
<body>
<div id="root"><%- initialHTML %></div>
<script>window.initialData = <%- JSON.stringify(initialData) %>;</script>
<script src="index.js"></script>
</body>
</html>
```


# JSX 文件

判断是否浏览器环境，若是，将 window.initialData 作为数据进行渲染：

```js
// ...
if (typeof window !== 'undefined') {
    ReactDOM.render(
        React.createElement(IndexPage, initialData),
        document.getElementById('root')
    );
}
```
