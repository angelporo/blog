---
title: Immutable.js意义以及使用场所
date: 2017-06-15 17:19:56
tags:
    - react
    - react-native
    - immutable
---

### [学习资料地址](https://juejin.im/post/5948985ea0bb9f006bed7472)

> Immutable Data 就是一旦创建，就不能再被更改的数据,对 Immutable 对象的任何修改或添加删除操作都会返回一个新的 Immutable 对象,

<!-- more -->

安装: `npm install immutable`

{% asset_img immuable.gif Immuable  %}

```
let foo = {a: {b: 1}};
let bar = foo;
bar.a.b = 2;
console.log(foo.a.b);  // 打印 2
console.log(foo === bar);  //  打印 true

// 使用 immutable.js 后
import Immutable from 'immutable';
foo = Immutable.fromJS({a: {b: 1}});
bar = foo.setIn(['a', 'b'], 2);   // 使用 setIn 赋值
console.log(foo.getIn(['a', 'b']));  // 使用 getIn 取值，打印 1

console.log(foo === bar);  //  打印 false
```

先放上官方`README`要解决的  (共享的可变状态是万恶之源)

上面的例子中已经看出来了, , , 着不是react设计初衷,

## Immutable 优点
1. Immutable 降低了 Mutable 带来的复杂度

	> 可变（Mutable）数据耦合了 Time 和 Value 的概念，造成了数据很难被回溯

2. 节省内存

	> Immutable.js 使用了 Structure Sharing 会尽量复用内存。没有被引用的对象会被垃圾回收。

```
import { Map} from 'immutable';
let a = Map({
  select: 'users',
  filter: Map({ name: 'Cam' })
})
let b = a.set('select', 'people');

a === b; // false

a.get('filter') === b.get('filter'); // true

```

3. Undo/Redo，Copy/Paste，甚至时间旅行这些功能做起来小菜一碟

    > 因为每次数据都是不一样的，只要把这些数据放到一个数组里储存起来，想回退到哪里就拿出对应数据即可，很容易开发出撤销重做这种功能。后面我会提供 Flux 做 Undo 的示例。

4. 并发安全传统的并发非常难做，因为要处理各种数据不一致问题，因此『聪明人』发明了各种锁来解决。但使用了 Immutable 之后，数据天生是不可变的，并发锁就不需要了。

然而现在并没什么卵用，因为 JavaScript 还是单线程运行的啊。但未来可能会加入，提前解决未来的问题不也挺好吗？

常用api示例:

```
//Map()  原生object转Map对象 (只会转换第一层，注意和fromJS区别)
immutable.Map({name:'danny', age:18})

//List()  原生array转List对象 (只会转换第一层，注意和fromJS区别)
immutable.List([1,2,3,4,5])

//fromJS()   原生js转immutable对象  (深度转换，会将内部嵌套的对象和数组全部转成immutable)
immutable.fromJS([1,2,3,4,5])    //将原生array  --> List
immutable.fromJS({name:'danny', age:18})   //将原生object  --> Map

//toJS()  immutable对象转原生js  (深度转换，会将内部嵌套的Map和List全部转换成原生js)
immutableData.toJS();

//查看List或者map大小
immutableData.size  或者 immutableData.count()

// is()   判断两个immutable对象是否相等
immutable.is(imA, imB);

//merge()  对象合并
var imA = immutable.fromJS({a:1,b:2});
var imA = immutable.fromJS({c:3});
var imC = imA.merge(imB);
console.log(imC.toJS())  //{a:1,b:2,c:3}

//增删改查（所有操作都会返回新的值，不会修改原来值）
var immutableData = immutable.fromJS({
    a:1,
    b:2，
    c:{
        d:3
    }
});
var data1 = immutableData.get('a') //  data1 = 1
var data2 = immutableData.getIn(['c', 'd']) // data2 = 3   getIn用于深层结构访问
var data3 = immutableData.set('a' , 2);   // data3中的 a = 2
var data4 = immutableData.setIn(['c', 'd'], 4);   //data4中的 d = 4
var data5 = immutableData.update('a',function(x){return x+4})   //data5中的 a = 5
var data6 = immutableData.updateIn(['c', 'd'],function(x){return x+4})   //data6中的 d = 7
var data7 = immutableData.delete('a')   //data7中的 a 不存在
var data8 = immutableData.deleteIn(['c', 'd'])   //data8中的 d 不存在
```
### 我认为最大的缺点就是容易和原生对象混淆

> 就是在开发的过程中,  有时候很你那分清楚是Immutable对象还是原生对象, Immutable中的Map和List虽然对应原生的Object和Array, 使用上还是有很大区别的, 主要还是API的不同带来的. 另外.Immuable每次修改会自动`return`新的Immutable, 还有就是使用的时候, 需要转换, 这个比较麻烦,  每次需要在和其他库配合的时候都需要转换

```
map.get('key') // Immuable对象取值
may.key  //原生对象取值
```

### 周边信息

两个 immutable 对象可以使用 `===` 来比较，这样是直接比较内存地址，性能最好。但即使两个对象的值是一样的，也会返回 `false`：

```
let map1 = Immutable.Map({a:1, b:1, c:1});
let map2 = Immutable.Map({a:1, b:1, c:1});
map1 === map2;             // false
```
为了直接比较对象的值，immutable.js 提供了 `Immutable.is` 来做『值比较』，结果如下：
```
Immutable.is(map1, map2);  // true
```
`Immutable.is` 比较的是两个对象的 `hashCode` 或 `valueOf`（对于 JavaScript 对象）。由于 immutable 内部使用了 Trie 数据结构来存储，只要两个对象的 `hashCode` 相等，值就是一样的。这样的算法避免了深度遍历比较，性能非常好。

后面会使用 `Immutable.is` 来减少 React 重复渲染，提高性能。

与 Object.freeze、const 区别

`Object.freeze` 和 ES6 中新加入的 `const` 都可以达到防止对象被篡改的功能，但它们是 shallowCopy 的。对象层级一深就要特殊处理了。

Cursor 的概念

这个 Cursor 和数据库中的游标是完全不同的概念。

由于 Immutable 数据一般嵌套非常深，为了便于访问深层数据，Cursor 提供了可以直接访问这个深层数据的引用。

```
import Immutable from 'immutable';
import Cursor from 'immutable/contrib/cursor';

let data = Immutable.fromJS({ a: { b: { c: 1 } } });
// 让 cursor 指向 { c: 1 }
let cursor = Cursor.from(data, ['a', 'b'], newData => {
  // 当 cursor 或其子 cursor 执行 update 时调用
  console.log(newData);
});

cursor.get('c'); // 1
cursor = cursor.update('c', x => x + 1);
cursor.get('c'); // 2
```

### 实践

1. 与 React 搭配使用，Pure Render

熟悉 React 的都知道，React 做性能优化时有一个避免重复渲染的大招，就是使用 `shouldComponentUpdate()`，但它默认返回 `true`，即始终会执行 `render()` 方法，然后做 Virtual DOM 比较，并得出是否需要做真实 DOM 更新，这里往往会带来很多无必要的渲染并成为性能瓶颈。

当然我们也可以在 `shouldComponentUpdate()` 中使用使用 deepCopy 和 deepCompare 来避免无必要的 `render()`，但 deepCopy 和 deepCompare 一般都是非常耗性能的。

Immutable 则提供了简洁高效的判断数据是否变化的方法，只需 `===` 和 `is` 比较就能知道是否需要执行 `render()`，而这个操作几乎 0 成本，所以可以极大提高性能。修改后的 `shouldComponentUpdate` 是这样的：

前几天在优化redux中的store使用immutable的时候发现一个问题,  那就是

```
import { is } from 'immutable';

shouldComponentUpdate: (nextProps = {}, nextState = {}) => {
  const thisProps = this.props || {}, thisState = this.state || {};

  if (Object.keys(thisProps).length !== Object.keys(nextProps).length ||
      Object.keys(thisState).length !== Object.keys(nextState).length) {
    return true;
  }

  for (const key in nextProps) {
    if (thisProps[key] !== nextProps[key] || ！is(thisProps[key], nextProps[key])) {
      return true;
    }
  }

  for (const key in nextState) {
    if (thisState[key] !== nextState[key] || ！is(thisState[key], nextState[key])) {
      return true;
    }
  }
  return false;
}
```
> 使用 Immutable 后，如下图，当红色节点的 state 变化后，不会再渲染树中的所有节点，而是只渲染图中绿色的部分:
{% asset_img immuable2.gif 使用后效果图  %}
你也可以借助 `React.addons.PureRenderMixin` 或支持 class 语法的[pure-render-decoratorhttps://github.com/felixgirault/pure-render-decorator]() 来实现。

setState 的一个技巧

React 建议把 `this.state` 当作 Immutable 的，因此修改前需要做一个 deepCopy，显得麻烦：
```
import '_' from 'lodash';

const Component = React.createClass({
  getInitialState() {
    return {
      data: { times: 0 }
    }
  },
  handleAdd() {
    let data = _.cloneDeep(this.state.data);
    data.times = data.times + 1;
    this.setState({ data: data });
    // 如果上面不做 cloneDeep，下面打印的结果会是已经加 1 后的值。
    console.log(this.state.data.times);
  }
}
```
使用 Immutable 后：
```javaScript
getInitialState() {
    return {
      data: Map({ times: 0 })
    }
  },
  handleAdd() {
    this.setState({ data: this.state.data.update('times', v => v + 1) });
    // 这时的 times 并不会改变
    console.log(this.state.data.get('times'));
  }
```
上面的 `handleAdd` 可以简写成：

```javaScript
handleAdd() {
    this.setState(({data}) => ({
      data: data.update('times', v => v + 1) })
    });
  }
```

### 与`Redux`配合使用遇到的坑
> 先说下`redux`中的单向数据流（View -> Action -> Middleware -> Reducer）, 项目中使用的redux,
>由于redux中内置的`combineReducers`和reducer中的`initialState`都会返回一个原生的Object对象, 所以配合`Imuutable`之后就会和原生Object搭配使用,  开发起来很不爽.
>幸运的是,  redux中并不排斥使用Immutable, 可以自己重写`combineReducers`或使用[ redux-immutablejs](https://github.com/indexiatech/redux-immutablejs)来提供支持

上面我们提到 Cursor 可以方便检索和 update 层级比较深的数据，但因为 Redux 中已经有了 select 来做检索，Action 来更新数据，因此 Cursor 在这里就没有用武之地了。

> [写的很不错,  原文地址](https://zhuanlan.zhihu.com/p/20295971?columnSlug=purerender)
