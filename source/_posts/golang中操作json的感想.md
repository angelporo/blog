---
layout: post
title: golang中操作json的感想
date: 2017-11-15 14:54:11
tags:
    - golang
    - json
categories: "算法"
---
看说说我自己的学习路线, 是那种看了基本语法和语言特征就先上手,  然后在动手撸代码的过程中来发掘语言中的奇妙, 然后就有了下面的感想.

> 问题来源于server端模拟client来集成`环信`来获取用户`token`的时候需要构建`request Body`来获取`token`

由于自己太愚昧别人很简单就可以理解的问题,  我狠狠的看了一个上午, 在加上对语法的不熟悉还有半天的各种实验

总结:
`encoding/json`中序列化和反序列化方法都是采用映射的方式来完成, 所以在构建结构体的时候需要把类型字段规定好.类似下面:
```golang
type Animal struct {
    Name  string `json:"name"` // "name"为json中key的字段
    Order string `json:"order"` // 同样, "order"也是json中key的字段
}
```

**要点**:
序列化之前和之后的**`struct`**数据**`type`**不能随便写, 必须对应下面的结构来:

```golang
bool, for JSON booleans
float64, for JSON numbers
string, for JSON strings
[]interface{}, for JSON arrays
map[string]interface{}, for JSON objects
nil for JSON null
```

一般使用到的基本就是序列化`Marshal`和反序列化`Unmarshal`这两个方法,  基本需求就差不多够了, 如果无法满足,  可以去官网查看其它`api`

### **`Marshal`**

将`struct`映射成`json`

需要注意的是声明`struct`的时候`tag`, 这里还有几种tag的注意事项:
- tag 是 "-"，表示该字段不会输出到 JSON.

- tag 中带有自定义名称，那么这个自定义名称会出现在 JSON 的字段名中，比如上面小写字母开头的 name.

- tag 中带有 "omitempty" 选项，那么如果该字段值为空，就不会输出到JSON 串中.

- 如果字段类型是 bool, string, int, int64 等，而 tag 中带有",string" 选项，那么该字段在输出到 JSON 时，会把该字段对应的值转换成 JSON 字符串.


官方例子:

```golang
package main

import (
    "encoding/json"
    "fmt"
)

// 这里需要映射的结构体内部属性必要是大写
type Animal struct {
    Name  string `json:"name"`  // tag中的"name"是json中的key值
    Order string `json:"order"` // 同样"order"是json的key值
}
func main() {
    var animals []Animal // 这里声明的是一个`slice`然而切片的没个`type`是`Animal`所以序列化后就是一个arrays -> [{}, {}]

    animals = append(animals, Animal{Name: "Platypus", Order: "Monotremata"})
    animals = append(animals, Animal{Name: "Quoll", Order: "Dasyuromorphia"})

    jsonStr, err := json.Marshal(animals)
    if err != nil {
        fmt.Println("error:", err)
    }

    fmt.Println(string(jsonStr))
}
```

打印结果:

```base
➜  blog git:(master) ✗
[{"name":"Platypus","order":"Monotremata"},{"name":"Quoll","order":"Dasyuromorphia"}]
```

### **`Unmarshal`**

将 JSON 串重新组装成结构体

函数签名

`func Unmarshal(data []byte, v interface{}) error`

官方例子:
```goalng
package main
import (
    "encoding/json"
    "fmt"
)
// 由于是反序列化, 所以不需要来写tag来规定json中的key
type Animal struct {
    Name  string
    Order string
}
func main() {
    var jsonBlob = []byte(`[
        {"Name": "Platypus", "Order": "Monotremata"},
        {"Name": "Quoll",    "Order": "Dasyuromorphia"}
    ]`)
    var animals []Animal
    // 由于被反序列化的是`slice`, 所以序列化之后就是一个Array类型array中的item类型是Animal
    err := json.Unmarshal(jsonBlob, &animals)
    if err != nil {
        fmt.Println("error:", err)
    }
    fmt.Printf("%+v", animals)
}
```
结果:

```golang
➜  imClientServer
[{Name:Platypus Order:Monotremata} {Name:Quoll Order:Dasyuromorphia}]
```

上面例子中可以看出，结构体字段名与 JSON 里的 KEY 一一对应.
例如 JSON 中的 KEY 是 Name，那么怎么找对应的字段呢？
- 首先查找 tag 含有 Name 的可导出的 struct 字段(首字母大写)

- 其次查找字段名是 Name 的导出字段

- 最后查找类似 NAME 或者 NAmE 等这样的除了首字母之外其他大小写不敏感的导出字段

**注意**：能够被赋值的字段必须是可导出字段！！

> 同时 JSON 解析的时候只会解析能找得到的字段，找不到的字段会被忽略，这样的一个好处是：当你接收到一个很大的 JSON 数据结构而你却只想获取其中的部分数据的时候，你只需将你想要的数据对应的字段名大写，即可轻松解决这个问题。


### 反序列化未知类型

前面说的是，已知要解析的类型，比如说，当看到 JSON arrays 时定义一个 golang 数组进行接收数据， 看到 `JSON objects` 时定义一个 map 来接收数据，那么这个时候怎么办？答案是使用 `interface{}` 进行接收，然后配合 `type assert` 进行解析，比如：

```golang
var f interface{} // 声明一个任意类型的f
b := []byte(`{"Name":"Wednesday","Age":6,"Parents":["Gomez","Morticia"]}`)
json.Unmarshal(b, &f)
for k, v := range f.(map[string]interface{}) { // 便利f的type
    switch vv := v.(type) {
    case string:
        fmt.Println(k, "is string", vv)
    case int:
        fmt.Println(k, "is int ", vv)
    case float64:
        fmt.Println(k, "is float64 ", vv)
    case []interface{}:
        fmt.Println(k, "is array:")
        for i, j := range vv {
            fmt.Println(i, j)
        }
    }
}
```
结果:
```base
➜  imClientServer go run test.go
Name is string Wednesday
Age is float64  6
Parents is array:
0 Gomez
1 Morticia
```
