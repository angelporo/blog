---
layout: post
title: golang中的writer和reader
date: 2017-11-07 18:00:13
tags:
    - golang
    - go基础
categories: "算法"
---

先来说说程序中的输入和输出

`c`里面有标准输出和输入还有错误处理, 还有非常重要的管道的, 以至于它非常好扩展.

那么`golang`中的`Write`和`Reader`的接口设计遵循了Unix的输入和输出, 一个程序的输出可以是另外一个程序的输入。他们的功能单一并且纯粹，这样就可以非常容易的编写程序代码，又可以通过组合的概念，让我们的程序做更多的事情。

### Unix的三种输入输出输入模式

```golang
var (
   Stdin  = NewFile(uintptr(syscall.Stdin), "/dev/stdin")
   Stdout = NewFile(uintptr(syscall.Stdout), "/dev/stdout")
   Stderr = NewFile(uintptr(syscall.Stderr), "/dev/stderr")
)
```
这三种标准的输入和输出都是一个`*File`，而`*File`恰恰就是同时实现了`io.Writer`和`io.Reader`这两个接口的类型，所以他们同时具备输入和输出的功能，既可以从里面读取数据，又可以往里面写入数据。

Go标准库的io包也是基于Unix这种输入和输出的理念，大部分的接口都是扩展了`io.Writer`和`io.Reader`，大部分的类型也都选择的实现了`io.Writer`和`io.Reader`这两个接口，然后把数据的输入和输出，抽象为流的读写，所以只要实现了这两个接口，都可以使用流的读写功能。

`io.Writer`和`io.Reader`两个接口的高度抽象，让我们不用再面向具体的业务，我们只关注，是读还是写，只要我们定义的方法函数可以接收这两个接口作为参数，那么我们就可以进行流的读写，而不用关心如何读，写到哪里去，这也是面向接口编程的好处。

### Reader和Writer接口

这两个高度抽象的接口，只有一个方法，也体现了Go接口设计的简洁性，只做一件事。

```golang
// Writer 接口包装了基本的 Write 方法。

// Write 将 len(p) 个字节从 p 中写入到基本数据流中。
// 它返回从 p 中被写入的字节数 n（0 <= n <= len(p)）以及任何遇到的引起写入提前停止的错误。
// 若 Write 返回的 n < len(p)，它就必须返回一个非nil的错误。Write 不能修改此切片的数据，即便它是临时的。

type Writer interface {
    Write(p []byte) (n int, err error)
}
```

这是`Wirter`接口的定义，它只有一个`Write`方法，接受一个`byte`的切片，返回两个值，n表示写入的字节数、err表示写入时发生的错误。

从其文档注释来看，这个方法是有规范要求的，我们要想实现一个io.Writer接口，就要遵循这些规则。

这些实现`io.Writer`接口的规则，所有实现了该接口的类型都要遵守，不然可能会导致莫名其妙的问题。

```golang
Reader 接口包装了基本的 Read 方法。

// Read 将 len(p) 个字节读取到 p 中。它返回读取的字节数 n（0 <= n <= len(p)） 以及任何遇到的错误。即使 Read 返回的 n < len(p)，它也会在调用过程中使用 p 的全部作为暂存空间。若一些数据可用但不到 len(p) 个字节，Read 会照例返回可用的东西， 而不是等待更多。

// 当 Read 在成功读取 n > 0 个字节后遇到一个错误或 EOF 情况，它就会返回读取的字节数。 它会从相同的调用中返回（非nil的）错误或从随后的调用中返回错误（和 n == 0）。 这种一般情况的一个例子就是 Reader 在输入流结束时会返回一个非零的字节数， 可能的返回不是 err == EOF 就是 err == nil。无论如何，下一个 Read 都应当返回 0, EOF。

// 调用者应当总在考虑到错误 err 前处理 n > 0 的字节。这样做可以在读取一些字节， 以及允许的 EOF 行为后正确地处理I/O错误。

// Read 的实现在 len(p) == 0 以外的情况下会阻止返回零字节的计数和 nil 错误， 调用者应将返回 0 和 nil 视作什么也没有发生；特别是它并不表示 EOF。

// 实现必须不保留 p。

type Reader interface {
    Read(p []byte) (n int, err error)
}
```

这是`io.Reader`接口定义，也只有一个`Read`方法，这个方法接受一个`byte`的切片，并返回两个值，一个是读入的字节数，一个是`err`错误。

- 从其注释文档看，`io.Reader`接口的规则更多。

- 从其注释文档看，`io.Reader`接口的规则更多。

- Read最多读取len(p)字节的数据，并保存到p。
- 返回读取的字节数以及任何发生的错误信息
- n要满足0 <= n <= len(p)
- n<len(p)时，表示读取的数据不足以填满p，这时方法会立即返回，而不是等待更多的数据
- 读取过程中遇到错误，会返回读取的字节数n以及相应的错误err
- 在底层输入流结束时，方法会返回n>0的字节，但是err可能时EOF，也可以是nil
- 在第6种(上面)情况下，再次调用read方法的时候，肯定会返回0,EOF
- 调用Read方法时，如果n>0时，优先处理处理读入的数据，然后再处理错误err，EOF也要这样处理
- Read方法不鼓励返回n=0并且err=nil的情况

### 例子

```golang
import(
    "bytes"
)

func main() {
    //定义零值Buffer类型变量b
    var b bytes.Buffer
    //使用Write方法为写入字符串
    b.Write([]byte("你好"))
    //这个是把一个字符串拼接到Buffer里
    fmt.Fprint(&b,",","http://www.flysnow.org")
    //把Buffer里的内容打印到终端控制台
    b.WriteTo(os.Stdout)
}
```
这个例子是拼接字符串到`Buffer`里，然后再输出到控制台，这个例子非常简单，但是利用了流的读写，`bytes.Buffer`是一个可变字节的类型，可以让我们很容易的对字节进行操作，比如读写，追加等。bytes.Buffer实现了io.Writer和io.Reader接口，所以我么可以很容易的进行读写操作，而不用关注具体实现。

`b.Write([]byte("你好"))`实现了写入一个字符串，我们把这个字符串转为一个字节切片，然后调用`Write`方法写入，这个就是`bytes.Buffer`为了实现`io.Writer`接口而实现的一个方法，可以帮我们写入数据流。

```golang

// Buffer 实现了Write接口
func (b *Buffer) Write(p []byte) (n int, err error) {
    b.lastRead = opInvalid
    m := b.grow(len(p))
    return copy(b.buf[m:], p), nil
}
```
以上就是`bytes.Buffer` 实现`io.Writer`接口的方法, 最终我们看到, 我们写入的切片会被写到`b.buf`里面, 这了`b.buf[m:]`拷贝其实就是追加的意思, 不会覆盖原来的数据.
从实现看，我们发现其实只有`b *Buffer`指针实现了`io.Writer`接口，所以我们示例代码中调用`fmt.Fprint`函数的时候，传递的是一个地址`&b`。

```golang
func Fprint(w io.Writer, a ...interface{}) (n int, err error) {
    p := newPrinter()
    p.doPrint(a)
    n, err = w.Write(p.buf)
    p.free()
    return
}
```

这是函数`fmt.Fprint`的实现,  他的功能就是为一个把数据`a`写入到一个`io.Write`接口实现了, 具体如何写入,  它是不关心的,  因为`io.Writer`会做的, 它只关心可以写入即可。`w.Write(p.buf)`调用`Wirte`方法写入。

最后的`b.WriteTo(os.Stdout)`是把最终的数据输出到标准的`os.Stdout`里，以便我们查看输出，它接收一个`io.Writer`接口类型的参数，开篇我们讲过`os.Stdout`也实现了这个`io.Writer`接口，所以就可以作为参数传入。

这里我们会发现，很多方法的接收参数都是`io.Writer`接口，当然还有`io.Reader`接口，这就是面向接口的编程，我们不用关注具体实现，只用关注这个接口可以做什么事情，如果我们换成输出到文件里，那么也很容易，只用把`os.File`类型作为参数即可。任何实现了该接口的类型，都可以作为参数。

除了`b.WriteTo`方法外，我们还可以使用`io.Reader`接口的`Read`方法实现数据的读取.
```golang
    var p [100]byte
    n,err:=b.Read(p[:])
    fmt.Println(n,err,string(p[:n]))
```
这是最原始的方法，使用Read方法，n为读取的字节数，然后我们输出打印出来。

因为byte.Buffer指针实现了io.Reader接口，所以我们还可以使用如下方式读取数据信息。

```golang
    data,err:=ioutil.ReadAll(&b)
    fmt.Println(string(data),err)
```

ioutil.ReadAll接口一个io.Reader接口的参数，表明可以从任何实现了io.Reader接口的类型里读取全部的数据。
```golang
func readAll(r io.Reader, capacity int64) (b []byte, err error) {
    buf := bytes.NewBuffer(make([]byte, 0, capacity))
    // If the buffer overflows, we will get bytes.ErrTooLarge.
    // Return that as an error. Any other panic remains.
    defer func() {
        e := recover()        
        if e == nil {
                    return
        }       
        if panicErr, ok := e.(error); ok && panicErr == bytes.ErrTooLarge {
                   err = panicErr
        } else {
                    panic(e)
        }
    }()
    _, err = buf.ReadFrom(r)    return buf.Bytes(), err
}
```
以上是ioutil.ReadAll实现的源代码，也非常简单，基本原理是创建一个byte.Buffer ,通过这个byte.Buffer的ReadFrom方法，把io.Reader里的数据读取出来，最后通过byte.Buffer的Bytes方法进行返回最终读取的字节数据信息。

整个流的读取和写入已经被完全抽象啦， io包的大部分操作和类型都是基于这两个接口，当然还有http等其他牵涉到数据流、文件流等的，都可以完全用io.Writer和io.Reader接口来表示，通过这两个接口的连接，我们可以实现任何数据的读写。
