---
layout: post
title: golang中interface作为函数参数
date: 2017-11-10 16:43:31
tags:
    - golang
    - interface{}
categories: "语法理解"
---

```golang
package main

import (
    "errors"
    "fmt"
)

// 定义item结构
type item struct {
    Name string
}
// item结构实现Stringer接口 在调用fmt.PrintIn时自动调用String方法
func (i item) String() string {
    return fmt.Sprintf("item name:%v", i.Name)
}

// 定义person结构
type person struct {
    Name string
    Sex  string
}
// person结构实现Stringer接口 在调用fmt.PrintIn时自动调用String方法
func (p person) String() string {
    return fmt.Sprintf("person name:%v sex:%v", p.Name, p.Sex)
}

// 这种情况一般是需要类型断言。类型断言接受一个接口值， 并从中提取指定的明确类型的值。
// 其语法借鉴自类型选择开头的子句，但它需要一个明确的类型， 而非 type 关键字：
// 说白了就是不明确传入type类型,  需要用interface来判断传入的具体type类型来做相应的操作
func Parse(i interface{}) interface{} {
    switch i.(type) {
    case string:
        return &item{
            Name: i.(string),
        }

    case []string:
        data := i.([]string)
        length := len(data)
        if length == 2 {
            return &person{
                Name: data[0],
                Sex:  data[1],
            }
        } else {
            return nil
        }

    default:
        panic(errors.New("Type match miss"))
    }

    return nil
}

func main() {
    p1 := Parse("apple").(*item)
    fmt.Println(p1)
    p2 := Parse([]string{"zhanghan", "man"}).(*person)
    fmt.Println(p2)// 调用的时候回自动调用String方法

    fmt.Println(p1.Name) // 调用的时候回自动调用String方法

    fmt.Println(p2.Name) // 调用的时候回自动调用String方法
    fmt.Println(p2.Sex) // 调用的时候回自动调用String方法
}
```

```base
// 运行结果为
➜  imClientServer
item name:apple
person name:zhanghan sex:man
apple
zhanghan
man
```

