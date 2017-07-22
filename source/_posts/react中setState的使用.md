---
title: react中setState的使用
date: 2017-07-08 16:29:38
tags:
    - react
    - setState
---

## 我认为`this.setState`是react中最常用的API了.

如果不明白`this.setState`是什么, [请移驾官网查看具体api](https://facebook.github.io/react/)

<!-- more -->

```javascript
class Timer extends React.Component {
  constructor(props) {
    super(props);
    this.state = {secondsElapsed: 0};
  }

  tick() {
    this.setState((prevState) => ({
      secondsElapsed: prevState.secondsElapsed + 1
    }));
  }

  componentDidMount() {
    this.interval = setInterval(() => this.tick(), 1000);
  }

  componentWillUnmount() {
    clearInterval(this.interval);
  }

  render() {
    return (
      <div>Seconds Elapsed: {this.state.secondsElapsed}</div>
    );
  }
}

ReactDOM.render(<Timer />, mountNode);
```

先看看`this.setState`是干什么的

{% asset_img this.setState.png setState流程图 %}

这里有必要说一下, `this.setState`是异步调用,
如果想要使用不同执行, 它的第二个参数是回调函数, 列队到达执行完修改state中状态后执行回调函数



