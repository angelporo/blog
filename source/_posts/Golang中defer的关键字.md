---
layout: post
title: Golang中defer, panic, recover
date: 2017-10-30 11:41:52
tags:
    - golang
    - defer
    - golang错误处理
categories: "算法"
---

刚入门`golang` 对一些关键是上面的理解还不是很好,  所以就除了看官网还是找前人理解的看法

## defer

defer 英文原意： vi. 推迟；延期；服从   vt. 使推迟；使延期。

Golang中的defer关键字实现比较特殊的功能，按照官方的解释，defer后面的表达式会被放入一个列表中，在当前方法`return`的时候，列表中的表达式就会被执行。一个方法中可以在一个或者多个地方使用defer表达式，这也是前面提到的，为什么需要用一个列表来保存这些表达式。在Golang中，defer表达式通常用来处理一些清理和释放资源的操作。

貌似看起来比较难懂，其实，如果你用过C#，一定记得那个用起来非常方便的using语句，defer可以理解成为了实现类似的功能。不过比起C#的using语句，defer的行为稍微复杂一些，想要彻底理解defer，需要了解Golang中defer相关的一些特性。

通过简单的例子, 我们就可以大致的了解`defer`的用法:

```golang
/*
 *  拷贝文件内容功能的函数
 *  srcName文件的内容拷贝到dstName文件中
 *  返回一个int和一个err
*/
func CopyFile( dstName, srcName string ) (written int64, err error) {
    src, err := os.Open(srcName)
    if err != nil {
        return
    }
    dst, err := os.Create(dstName)
    if err != nil {
        return
    }
    written, err = io.Copy(dst, src)
    dst.Close()
    src.Close()
    return
}
```

上面例子中看似没什么问题,  不过`Golang`中的资源需要释放, 假如`os.Create`方法的调用出现了错误,  下面的语句就直接return, 导致这两个打开的文件没有机会被释放, 这个时候, `defer`就派上用场了.

```golang
/**
 * 这个是使用defer改进后的例子
**/
func CopyFile(dstName, srcName string) (written int64, err error) {
    src, err := os.Open(srcName)
    if err != nil {
        return
    }
    defer src.Close() // 使用defer src文件被释放再调用
    dst, err := os.Create(dstName)
    if err != nil {
        return
    }
    defer dst.Close() // 使用defer src文件被释放再调用
    return io.Copy(dst, src)
}
```

改进的代码中两处都使用了`defer`表达式, 表达式的内容就是关闭文件, 前面介绍过, 虽然表达式的具体行为是关闭文件,  但是并不会被马上执行, 两个表达式都会被放入一个list中, 等待被调用, 先卖个关子，这个list可以看作是一个栈(stack)的结构，是一个后进先出的栈。

知道`defer`的基本用法,  我们继续深入了解一下`defer`的一些特性

- ##`defer`##表达式中的变量值再defer表达式被定义是就已经明确

```golang
func a() {
    i := 0
    defer fmt.PrintIn(i)
    i++
    return
}
```

上面的这段代码，defer表达式中用到了i这个变量，i在初始化之后的值为0，接着程序执行到defer表达式这一行，表达式所用到的i的值就为0了，接着，表达式被放入list，等待在return的时候被调用。所以，后面尽管有一个i++语句，仍然不能改变表达式 fmt.Println(i)的结果。

所以，程序运行结束的时候，输出的结果是0而不是1。

- `defer`表达式的调用顺序是按照先进后出的方式

```golang
func b() {
    defer fmt.Print(1)
    defer fmt.Print(2)
    defer fmt.Print(3)
    defer fmt.Print(4)
}
```

前面已经提到过，defer表达式会被放入一个类似于栈(stack)的结构，所以调用的顺序是后进先出的。所以，上面这段代码输出的结果是4321而不是1234。在实际的编码中应该主意，程序后面的defer表达式会被优先执行。

- defer表达式中可以修改函数中的命名返回值

Golang中的函数返回值是可以命名的，这也是Golang带给开发人员的一个比较方便特性。
```golang
func c() (i int) {
    defer func() { i++ }()
    return 1
}
```

上面的示例程序，返回值变量名为i，在defer表达式中可以修改这个变量的值。所以，虽然在return的时候给返回值赋值为1，后来defer修改了这个值，让i自增了1，所以，函数的返回值是2而不是1。

理解了defer的三个特性，用到defer的时候就能心中有数了。

## panic

一般配合recover来使用,  但是不推荐使用这个方法来对异常处理,  因为标准库和第三方库基本都会把`err`return出去,  然后供给开发者做处理

golang中没有`try catch`这样想弱类型语言的错误处理方式是不能用的,而是使用`panic`和`recover`来处理异常,这也和他的语言设计场景有关系,  毕竟是系统级的高性能层面的, 这种精准错误处理应该减少那种后遗症bug。

panic 英文原意：n. 恐慌，惊慌；大恐慌  adj. 恐慌的；没有理由的  vt. 使恐慌  vi. 十分惊慌

panic 是一个内置函数，当一个函数 F 调用 panic，F 的执行就会停止，F 中 deferred 函数调用会被执行，然后 F 返回控制到它的调用者。这个过程会沿着调用栈执行下去，直到当前 goroutine 中的所有函数返回，然后程序 crash。出现 panic 是因为：

调用了 panic 函数
出现了运行时错误（例如，数组越界访问）
recover 是一个内置函数，用于恢复一个 panicking goroutine 的控制。需要注意的是，recover 只能使用在 deferred 函数中。如果当前的 goroutine panicking，recover 调用将会捕获 panic 传递的值并且恢复正常的执行。看一个例子：

```golang
package main

import "fmt"

func main() {
  f() // 入口函数中调用f函数
  fmt.Println("Returned normally from f.")
}

func f() {
  defer func() {  // defer表达式的自调用匿名函数
    if r := recover(); r != nil {
      fmt.Println("Recovered in f", r)
    }
  }()
  fmt.Println("Calling g.")
  g(0)
  fmt.Println("Returned normally from g.")
}

func g(i int) {
  if i > 3 {
    fmt.Println("Panicking!")
    panic(fmt.Sprintf("%v", i))
  }
  defer fmt.Println("Defer in g", i)
  fmt.Println("Printing in g", i)
  g(i + 1)
}
```
