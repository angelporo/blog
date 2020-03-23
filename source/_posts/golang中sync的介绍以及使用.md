---
layout: post
title: golang中sync的介绍以及使用
date: 2018-01-25 14:48:24
tags:
    - golang
    - sync
    - linux内核中自带锁
    - golang异步锁
categories: "算法"
---

## 前言
有些不是科班出身的有时候学习的时候就会看不懂一些问题的本质,  今天就说说golang中也是很多程序设计的锁的作用, 具体参考`golang`来介绍锁的作用, 当然这个锁和数据库的锁是类似的.

先来个例子

```go
a := 0
// goroutine 1
go func () {
    for i := 0; i< 10000; i++ {
        a += 1
    }
}()

// goroutine 2
go func () {
    for i := 0; i< 10000; i++ {
        a += 1
    }
}()

// goroutine 1 和goroutine 2 并发执行
time.Sleep(time.Second) // 等待goroutine执行完成
fmt.Println(a)


// 执行结果
/*
➜  src go run test.go
102629
*/
```

上面代码可以很简单的看出, 正确的应该输出`200000`, 但是实际情况是小于`200000`的.
问题在于`goroutine 1`不知道`goroutine 2`中a的变化, 导致最终结果错误, 这时候就需要使用锁.

## 使用场景

上面代码中可以看出

在进行并发编程的时候, 通过共享内存的方式进行通信, 这时候可能会导致资源的竞争, 最终导致出现数据报错.

## 锁介绍
> `sync`包提供了基本的同步基元，如互斥锁。除了Once和WaitGroup类型，大部分都是适用于低水平程序线程，高水平的同步使用channel通信更好一些。

`golang`的`sync`实现了两种类型的锁:
> - sync.Mutex
> - sync.RWMutex

通过代码字面量信息就可以知道`sync.RWMutex`是基于`sync.Mutex`实现的.其中的只读锁的实现使用了类似引用计数的方式


### 互斥锁`sync.Mutex`
api
```go
var mutex sync.Mutex
mutex.Lock() // 加锁
// ...
mutex.UnLock() // 解锁
```

接着我们通过`symc`这个包, 来解决上面并发执行的问题

```go
var mutex sync.Mutex
a := 0
// goroutine1
go func() {
   for i := 0; i < 100000; i++ {
      mutex.Lock() // 上互斥锁
      a += 1
      mutex.Unlock() // 解互斥锁
   }
}()
// goroutine2
go func() {
   for i := 0; i < 100000; i++ {
      mutex.Lock()
      a += 1
      mutex.Unlock()
   }
}()
time.Sleep(time.Second)
fmt.Println(a)
```

这下执行结果就是`200000`

接下来我们使用例子的方式在看看
### 读写锁`sync.RWMutex`

通常情况下互斥锁已经能满足很多的应用场景了, 不过, 互斥锁比较耗费资源, 会拖慢执行效率. 在一些`读取操作谢雨写入操作`的场景下, 互斥锁就不大适应了,  这个时候我们就需要读写锁, 顾名思义, 这个就是为了避免并发执行读写发生冲突设计的锁.

api
```go
var rwmutex sync.RWMutex
rwmutex.Lock() // 加写锁
rwmutex.Unlock() // 解写锁
rwmutex.RLock() // 加读锁
rwmutex.RUnlock() // 解读锁
```

让我们看看两种锁的执行时间差异
```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func main() {
	mutex()
	rwmutex()
}

// 使用互斥锁
func mutex() {
	var mutex sync.Mutex
	var wg sync.WaitGroup

	bt := time.Now()
	a := 0
	b := 0
	wg.Add(9) // Add方法向内部计数加上delta，delta可以是负数；如果内部计数器变为0，Wait方法阻塞等待的所有线程都会释放，如果计数器小于0，方法panic。注意Add加上正数的调用应在Wait之前，否则Wait可能只会等待很少的线程。一般来说本方法应在创建新的线程或者其他应等待的事件之前调用。
	// 读操作
	for i := 1; i < 10; i++ {
		go func() {
			for i := 0; i < 100000; i++ {
				mutex.Lock()
				b = a
				mutex.Unlock()
			}
			wg.Done() // Done方法减少WaitGroup计数器的值，应在线程的最后执行
		}()
	}

	wg.Add(1)
	// 写操作
	go func() {
		for i := 0; i < 100; i++ {
			mutex.Lock()
			a += 1
			mutex.Unlock()
		}
		wg.Done()
	}()

	wg.Wait()
	et := time.Now().Sub(bt)
	fmt.Println("mutex time:", et.String())
}

// 使用读写锁
func rwmutex() {
	var rwmutex sync.RWMutex
	var wg sync.WaitGroup

	bt := time.Now()
	a := 0
	b := 0
	wg.Add(9)
	// 读操作
	for i := 1; i < 10; i++ {
		go func() {
			for i := 0; i < 100000; i++ {
				// 上读锁，多个goroutine可同时获取读锁，这里不会阻塞
				// 但读锁会阻塞其他写锁，直到读锁释放
				rwmutex.RLock()
				b = a
				rwmutex.RUnlock()
			}
			wg.Done()
		}()
	}

	wg.Add(1)
	// 写操作
	go func() {
		for i := 0; i < 100; i++ {
			// 上写锁，会阻塞其他的读锁和写锁，直到写锁释放
			rwmutex.Lock()
			a += 1
			rwmutex.Unlock()
		}
		wg.Done()
	}()

	wg.Wait()
	et := time.Now().Sub(bt)
	fmt.Println("rwmutex time:", et.String())
}
```

执行结构
```base
➜  src go run test.go
mutex time: 109.218727ms
rwmutex time: 38.589895ms
```

可以看出差距还是很大的
