---
layout: post
title: go中的interface理解
date: 2017-10-21 16:35:08
tags:
    - go
    - interface
categories: "算法"
---

刚开始学习`go`看了语法之后觉得`interface`理解不了, 然后也不清楚什么情况下来使用这个接口, 所以在网上就翻来翻去, 找到一个比较亲民的理解, 
```javascript
// TODO: channel和interface使用场景
```

## 在之前先看一下`func`的一些基础定义
在golang中定义函数的方式
```golang
func (p myType ) funcName ( a, b int , c string ) ( r , s int ) {
    return
}
```
- 关键字——func
- 方法名——funcName
- 入参——— a,b int,b string
- 返回值—— r,s int
- 函数体—— {}

这里具体说一下**`(p myType)`**
在`Go`中通过给函数标明所属类型，来给该类型定义方法, 也就是通过函数来给类型扩展方法, 上面的`p myType`表示给myType扩展funcName方法`p myType`不是必须的。如果没有，则纯粹是一个函数，通过包名称访问。packageName.funcationName

## `interface`
如果一只鸟长得像鸭子，走起路来像鸭子，叫起来也像鸭子，那么就把这只鸟叫做鸭子；

- 如果一个结构体绑定的方法包含接口的所有方法,即认为实现了该接口
- 将对象赋值给接口时,会发生拷贝,而接口的存储是指向这个复制的指针,复制的无法修改原来状态,也无法获取指针

golang中的interface就是上面这个意思，如果你定义了一个struct，它里面的方法和属性都和interface中的一样，那么可以说，这个struct实现这个interface,上代码

```go
package main

import (
	"fmt"
)

type s struct {
    //定义一个s类型，有一个属性i是int的
	i int
}

func (this *s) Get() int {
    //Get方法获得i属性
	return this.i
}

func (this *s) Put(v int) {
    //Put方法设置i属性
	this.i = v
}

type I interface {
    //定义一个接口类型，里面有Get方法与Put方法
	Get() int
	Put(int)
}

func main() {
	var S s
    //申请一个S变量，他是s类型的值
	f(&S)
}

func f(my I) {
    //这里的my保存了接口类型的值，因为s实现了I，所以传递的my虽然是个I类型，但是可以当作s类型来使用
	my.Put(999)
	fmt.Println(my.Get())
}
```

上面例子中f()虽然需要一个I类型的值,  但是s类型已经现式implement了I类型的接口(s类型实现了I接口要求的所有函数), 所以同样传入取地址的S`&S`变量同样可以调用方法

这种方式有一个专业的说法(非侵入式接口)

非侵入式接口一个很重要的好处就是去掉了繁杂的继承体系，我们看许大神在《go语言编程》一书中作的总结：
- Go语言的标准库，再也不需要绘制类库的继承树图。你一定见过不少C++、 Java、 C# 类库的继承树图。在Go中，类的继承树并无意义，你只需要知道这个类实现了哪些方法，每个方法是啥含义就足够了.
- 实现类的时候，只需要关心自己应该提供哪些方法，不用再纠结接口需要拆得多细才 合理。接口由使用方按需定义，而不用事前规划.
- 不用为了实现一个接口而导入一个包，因为多引用一个外部的包，就意味着更多的耦 合。接口由使用方按自身需求来定义，使用方无需关心是否有其他模块定义过类似的接口.


首先golang不支持面向对象思想, 它只能使用`interface`来模拟继承, 本质就是组合, 也可以通过`interface`来实现多态

**上面是对`interface`的类型理解**

**下面说一下在函数中的使用**:

在编写程序的时候会有传入函数的参数不确定的情况,或者`return`不确定参数, 那么我们就可以使用**空接口**来代替你所使用到的类型, 比如`int string float64 []interface error`等不同的类型
如果返回不同的类型就会导致编译出错, 但是你还需要使用不同的类型

```golang
// ToStr传入空接口用来代替任何类型
func ToStr(i interface{}) string {
    // 使用fmt中的Sprintf函数来把Interface转换成了String
    return fmt.Sprintf("%v", i)
}
ToStr(1)
ToStr(float64(1.234254354))
// 传入不同类型 编译不会报错
```
打印结果:
```base
➜  imClientServer go run test.go
str: 1, str2: 1.234254354%
```

同样传入参数和返回内容都可以使用`interface{}`来代替任何类型

```golang
// 传入任何类型 返回slice任何类型
func ToSlice(arr interface{}) []interface{} {
  v := reflect.ValueOf(arr)
  if v.Kind() != reflect.Slice {
    panic("toslice arr not slice")
  }
  l := v.Len()
  ret := make([]interface{}, l)
  for i := 0; i < l; i++ {
    ret[i] = v.Index(i).Interface()
  }
  return ret
}
```
