---
layout: post
title: golang中并发执行和通道传参以及并发中错误处理
date: 2017-11-27 09:55:18
tags:
    - chan
    - golang通道
    - panic
    - recover
categories: "golnag并发执行"
---

## `panic`和`recover`捕获运行中的错误
先来看看`panic`和`recover`的配合使用:

`panic`虽然官方不推荐使用, 但是有些业务需求必须要使用它来捕获**运行**中错误.

> go中可以抛出一个panic的异常，然后在defer中通过recover捕获这个异常，然后正常处理
> 在一个主进程，多个go程处理逻辑的结构中，这个很重要，如果不用recover捕获panic异常，会导致整个进程出错中断
```golang
package main
import "fmt"
func main() {
    // 程序在错误的时候才会调用defer定义的函数
	defer func() {     //必须要先声明defer，否则不能捕获到panic异常 不知道defer的可以查阅相关博客
		fmt.Println("c")
		if err := recover(); err != nil {
			fmt.Println(err)    //这里的err其实就是panic传入的内容，55
		}
		fmt.Println("d")
	}()
	f()
}


func f() {
	fmt.Println("a")
	panic(55)
	fmt.Println("b")

	fmt.Println("f")
}
```

运行结果
```base
➜  imClientServer git:(login) ✗ go run test.go
a
c
55
d
```
`panic`和`recover`只是稳固一下,是为了不懂得童鞋可以继续下面的实例



## 完整的一个`多协程`和`通道`的例子,
下面讲说了一个很好的例子, 其中有携程执行中出错然后对错误的处理方式,不过具体的错误还需要在业务需求做出相对应的处理.

有几个地方需要注意：for i + 协程时如果协程使用 i ，那么需要增加 i:=i 来防止多协程冲突；实际执行任务时需要用一个函数包起来，防止单个任务panic造成整个程序崩溃。

```golang
package main
import (
    "sync"
    "fmt"
)

/*
一个标准的协程+信道实现

*/

func main() {

    taskChan := make(chan int)
    TCount := 10
    var wg sync.WaitGroup //创建一个sync.WaitGroup

    // 产生任务
    go func() {
        for i := 0; i < 1000; i++ {
            taskChan <- i
        }
        // 全部任务都输入后关闭信道，告诉工作者进程没有新任务了。
        close(taskChan)
    }()

    // 告诉 WaitGroup 有 TCount 个执行者。
    wg.Add(TCount)
    // 启动 TCount 个协程执行任务
    for i := 0; i < TCount; i++ {

        // 注意：如果协程内使用了 i，必须有这一步，或者选择通过参数传递进协程。
        // 否则 i 会被 for 所在的协程修改，协程实际使用时值并不确定。
        i := i

        go func() {

            // 协程结束时报告当前协程执行完毕。
            defer func() { wg.Done() }()

            fmt.Printf("工作者 %v 启动...\r\n", i)

            for task := range taskChan {

                // 建立匿名函数执行任务的目的是为了捕获单个任务崩溃，防止造成整个工作者、系统崩溃。
                func() {

                    defer func() {
                        err := recover()
                        if err != nil {
                            fmt.Printf("任务失败：工作者i=%v, task=%v, err=%v\r\n", i, task, err)
                        }
                    }()

                    // 故意崩溃，看看是不是会造成整个系统崩溃。
                    if task%100==0{
                        panic("故意崩溃啦")
                    }

                    // 这里的 task 并不需要通过参数传递进来。
                    // 原因是这里是同步执行的，并不会被其它协程修改。
                    fmt.Printf("任务结果=%v ，工作者id=%v, task=%v\r\n",task*task,i,task)
                }()
            }

            fmt.Printf("工作者 %v 结束。\r\n", i)
        }()
    }

    //等待所有任务完成
    wg.Wait()
    print("全部任务结束")
}

```
