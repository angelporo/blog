---
title: react中高阶组件的使用以及技巧
date: 2017-07-24 16:33:04
tags:
    - react-native
    - react
    - 高阶组件
---

{% asset_img react-hoc.png 高阶组件抽象表单功能而非UI %}

[写的不错, 先转载, 之后在写自己的学习成果](https://zhuanlan.zhihu.com/p/27434557)

<!-- more -->

这篇文章完全就是为了更好的理解React中的高阶组件,

### 高阶组件和高阶函数的定义

高阶函数
接受函数作为参数, 或者输入另一个函数的一类函数,  被称为高阶函数.

对于高阶组件
接受一个组件作为参数,  这个组件,  可以是纯函数组件也可以`class extends Component`组件. 输出一个新的React组件的组件,  更通俗地描述为，**高阶组件**通过包裹（wrapped）被传入的React组件，经过一系列处理，最终返回一个相对增强（enhanced）的React
高阶组件是`React`中复用组件逻辑的一种进阶技巧,  他是一种技巧,  并不是什么特别高大上的一种API,  而是一种`React`组件设计理念,  众多的React库已经表明了, 这一设计的价值.例如`React-Redux`

### 实现一个简单的高阶组件

下面我们来实现一个简单的高阶组件(函数),  它接受一个React组件,  然后返回一个增强版的组件

```javascript

export default function WithHeader(WrappedComponent) {
  return class HOC extends Component {
    render () {
      return (
        <div>
          <div className="demo-header">
            这里是标题
          </div>
          <WrappedComponent {...this.props} />
        </div>
      );
    }
  };
}
```

大家可以看出这个组件的效果,  就是给传入的组件添加了一个标题,  但是这个效果我们也可以用其他的方法来实现, 下面说为什么这样设计,  也就是高阶函数的设计优点.

接下来, 我们来使用这个高阶组件,  用来强化之前被作为参数传入的组件

```javascript
@WithHeader
export class Demo extends Component {
  render () {
    return (
      <div>
        这是一个普通组件
      </div>
    );
  }
}
```
在这里使用了ES7里的`decorator`，来提升写法上的优雅，但是实际上它只是一个语法糖，下面这种写法也是可以的。
```javascript
const EnhanceDemo = WithHeader(Demoe);
```

随后，观察React组件树发生了什么变化，如图所示，可以发现Demo组件被HOC组件包裹起来了，符合了高阶组件的预期，即组件是层层包裹起来的，如同洋葱一样。

{% asset_img hoc1.png 被强化的高阶组件 %}

因为高阶组件最终`return`了一个明为`HOC`的组件,
所以在多次使用高阶组件之后, 在调试的时候会看到一大推`HOC`组件, 所以要做一个点小的优化, 就是在使用高阶组件包裹后,  应该保留原有的名称,  这样调试才会有好.

我们改写一些上面的高阶组件代码, 增加了`getDisplayName`函数以及静态属性`displayName`, 这个静态属性就是为了查看组件名称的. 这是在去观察呢`DOM Tree`

```javascript
function getDisplayName(component) {
  return component.displayName || component.name || 'Component';
}

export default function ( WrappedComponent ) {
  return class HOC extends Component {
    static displayName = `HOC(${getDisplayName(WrappedComponent)})`
    render () {
      return (
        <div>
          <div>这里是标题</div>
          <WrappedComponent { ...this.props} />
        </div>
      );
    }
  };
}
```
这样,  给每个需要增强的组件添加一个静态属性查看组件的名字

{% asset_img hoc2.png 添加静态名称属性 %}


#### 总结
> 上面这个例子里高阶组件只做了一件事, 就是给传入组件添加一个标题样式 ,  这个组件可以是任何一个添加次逻辑的组件上面,  值需要被高阶组件装饰即可.
> 由此可以看出, 高阶组件的主要功能是封装并抽离组件的通用逻辑, 让此部分逻辑在组件之间更好的复用.

### 高阶组件的进阶用法

#### 1. 组件参数

还是上面这个例子为例,  词高阶函数仅仅只是展示了标题内容`这里是标题`这个名称, 但是为了更好的抽象, 标题内容可以作为参数来获取, 如下面方式调用

```javascript
// 如果传入参数, 则传入的参数将作为组件的标题呈现
@WithHeader('Demo')
export default class Demo extends Component {
  render () {
    return (
      //...
    );
  }
}
```
`WithHeader`需要改写层如下形式, 它接受一个参数, 然后反悔一个高阶组件(函数).
```javascript
export default function (title) {
  return function (WrappedComponent) {
    return class HOC extends Component {
      render () {
        return (
          <div>
            <div className="demoe-header">
              { title ? title : '这里是标题'}
            </div>
            <WrappedComponent { ...this.props } />
          </div>
        );
      }
    };
  };
}
```
配合上es6写法可以更加简介
```javascript
export const MyComponent = (title) => (WrappedComponent) => class HOC extends Component {
  render () {
    return (
      <div>
        <div className="demoe-header">
          { title ? title : '这里是标题'}
        </div>
        <WrappedComponent { ...this.props } />
      </div>
    );
  }
}
```
调用:
```javascript
MyComponent('参数传入标图')(Demo);
```
上面这个高阶组件能够对`WrappedComponent`组件内的`props`进行操作, 提取`WrappedComponent`中的`state`以及使用其他原属来包裹`WrappedComponent`。`Props Proxy` 作为一层代理，会发生隔离，因此传入 `WrappedComponent` 的 `ref` 将无法访问到其本身(!这里我还没有理解为什么)，需在 `Props Proxy` 内完成中转，具体可参考以下代码，`react-redux` 也是这样实现的。


进阶:

```javascript
function getDisplayName(component) {
  return component.displayName || component.name || 'Component';
}

export const ppHOC = WrappedComponent => class PP extends Component {
  // 添加静态显示名称属性
  static displayName = `HOC(${WrappedComponent.displayName})`;
  getWrappedInstance () {
    return this.wrappedInstance;
  }
  // 实现ref的访问
  setWrappedInstance (ref) {
    this.wrappedInstance = ref;
  }
  render () {
    return (
      <WrappedComponent
          {
          // 注意这里是一个对象
          ...this.props
        ref: this.setWrappedInstance.bind(this)
          }
        />
    );
  }
}
@ppHOC
class Example extends React.Component {
  static displayName = 'Example';
  handleClick() { ... }
  ...
}
class App extends React.Component {
  handleClick() {
    this.refs.example.getWrappedInstance().handleClick();
  }
  render() {
    return (
      <div>
        <button onClick={this.handleClick.bind(this)}>按钮</button>
        <Example ref="example" />
      </div>
    );
  }
}
```

归纳:
#### HOC的使用范围对比
HOC 范式 compose(render)(state) 与父组件（Parent Component）的范式 render(render(state))，如果完全利用 HOC 来实现 React 的 implement，将操作与 view 分离，也未尝不可，但却不优雅。HOC 本质上是统一功能抽象，强调逻辑与 UI 分离。但在实际开发中，前端无法逃离 DOM ，而逻辑与 DOM 的相关性主要呈现 3 种关联形式：

- 与 DOM 相关，建议使用父组件，类似于原生 HTML 编写
- 与 DOM 不相关，如校验、权限、请求发送、数据转换这类，通过数据变化间接控制 DOM，可以使用 HOC 抽象
- 交叉的部分，DOM 相关，但可以做到完全内聚，即这些 DOM 不会和外部有关联，均可

HOC 适合做 DOM 不相关又是多个组件共性的操作。如 Form 中，validator 校验操作就是纯数据操作的，放到了 HOC 中。但 validator 信息没有放到 HOC 中。但如果能把 Error 信息展示这些逻辑能够完全隔离，也可以放到 HOC 中（可结合下一小节 Form 具体实践详细了解）。

```javascript
connect(props => ({
  usersFetch: `/users?status=${props.status}&page=${props.page}`,
  userStatsFetch: { url: `/users/stats`, force: true }
}))(UsersList)
```

### HOC 的具体实践
HOC 在真实场景下的运行非常多，之前笔者在 基于Decorator的组件扩展实践 一文中也提过使用高阶组件将更细粒度的组件组合成 Selector 与 Search。结合精读文章，这次让我们通过 Form 组件的抽象来表现 HOC 具有的良好扩展机制。

Form 中会包含各种不同的组件, 常用的有Input, Selector, Chackbox, 等等, 也会有根据业务需求加入自定义组件. form 灵活多变,  从功能上面, 表单验证可能未单组件校验, 也可能为全表单校验, 可能未常规校验, 比如: 非空, 输入限制, 也可能需要与服务端配合, 甚至需要根据业务特点进行定制, 从UI上看, 检验结果显示的位置, 可能在组件下方, 也可能在组件上方.

如果直接漏写form表单, 无意识机械而又重复的, 将`Form`中组件的`validator`, 把value, validator 产生的error信息储存到`state`或`redux store`中, 然后在`view`层完成显示. 这样的具体操作可能都是相同的, 可以进行复用, 只是我们面对的是不同的组件, 不同的validator, 不同的`view`而已. 对于Form而言, 既要满足通用, 又要满足部分个性化的需求, 以往单纯的配置话只会让使用瑜伽繁琐, 我们只需要休想的是Form功能而非UI,  因此通过HOC正对Form的功能进行提取就成为了必然.

至于 HOC 在 Form 上的具体实现，首先将表单中的组件（Input、Selector...）与相应 validator 与组件值回调函数名（trigger）传入 Decorator，将 validator 与 trigger 相绑定。Decorator 完成
了各种不同组件与 From 内置 Store 间 value 的传递、校验功能的抽象，即精读文章中提到 Props Proxy 方式的其中两种作用：提取state 与 操作props
[form库](https://github.com/react-component/form)的实现方式就是这种
```javascript
import { createForm } from 'rc-form';

class Form extends React.Component {
  submit = () => {
    this.props.form.validateFields((error, value) => {
      console.log(error, value);
    });
  }

  render() {
    const { getFieldError, getFieldDecorator } = this.props.form;
    const errors = getFieldError('required');
    return (
      <div>
        {getFieldDecorator('required', {
          rules: [{ required: true }],
        })(<Input />)}
        {errors ? errors.join(',') : null}
        <button onClick={this.submit}>submit</button>
      </div>
    );
  }
}

export createForm()(Form);
```

觉得有必要说一下高阶函数和函数传参的区别,  同样,  函数传参也同样能达到效果, 为什么高阶函数就使用起来很方便


> [参考文章](https://zhuanlan.zhihu.com/p/27434557)
> [参考文章](https://yq.aliyun.com/articles/149115?utm_content=m_27063)
