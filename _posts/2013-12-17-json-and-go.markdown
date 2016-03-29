---
layout: post
title: 'json and golang'
date: 2013-12-17
author: Pengfei.X
version: 0.1
categories: [golang, ]
excerpt: The json package only accesses the exported fields of struct types (those that begin with an uppercase letter). Therefore only the the exported fields of a struct will be present in the JSON output.
---


JSON（javascript object Notation）是轻量级的数据交换格式 它和Javascript中的对象和数组语法上类似 JSON目前广泛用于前后台数据交互 [JSON的官网](json.org)提供了更多的介绍
    

## Encoding ##

使用Marshal函数来将数据编码成JSON数据, Marshal原型如下:

    func Marshal(v interface{}) ([]byte, error)


下面这个例子使用Marshal来把Go中的struct序列化成JSON字符串:

    type Message struct {
        Name string
        Body string
        Time int64
    }

    m := Message{"Alice", "Hello", 1294706395881547000}
    b, err := json.Marshal(m)

    b == []byte(`{"Name":"Alice","Body":"Hello","Time":1294706395881547000}`) // this will be true
    

## Decoding ##

可以使用Unmarshal来decode JSON数据, Unmarshal的原型如下: 

    func Unmarshal(data []byte, v interface{}) error 

首先创建一个变量, decode后的JSON数据将被存到这个变量中去 

    var m Message

调用`json.Unmarshal` 

    err := json.Unmarshal(b, &m)
    /*
    m = Message{
            Name: "Alice",
            Body: "Hello",
            Time: 1294706395881547000,
    }
    */
   

## Unmarshal如何把JSON数据存到结构体对应的字段? ##

如果有一个JSON的Key为"Foo", Unmarshal怎样找到结构体中对应的域呢？ Unmarshal会按照下面的
次序来比遍历struct：

1.  带标记`Foo`的exported域 比如：

        struct {
            Microsec  uint64 "Foo"
        }

2.  struct中Foo字段

        struct {
            Foo uint64
        }

3.  其它可以被外部访问的 忽略大小写的字段 比如 FOO FOo等

        struct {
            FoO uint64
        }
    

## 如果JSON的结构与定义的Go结构体不能匹配 会出现什么样的状况呢？ ##

    b := []byte(`{"Name":"Bob","Food":"Pickle"}`)
    var m Message
    err := json.Unmarshal(b, &m)

Unmarshal只会把它可以识别出来的那些域反序列化 存放到m 像上面代码中Food字段就会被忽略, 当然了, 那些不能被外部访问的字段肯定是彻底被无视了. 有的时候, 我们并不知道JSON的结构是怎么样的, 哎, 头疼阿!
   

## Generic JSON with interface{} ##

空接口可以被当做是通用容器：

    var i interface{}
    i = "a string"
    i = 2011
    i = 2.777

通过类型断言获取类型信息：

    r := i.(float64) 
    fmt.Println("the circle's area", math.Pi*r*r)

或者也可以用`switch & case`：

    switch v := i.(type) {
    case int:
        fmt.Println("twice i is", v*2)
    case float64:
        fmt.Println("the reciprocal of i is", 1/v)
    case string:
        h := len(v) / 2
        fmt.Println("i swapped by halves is", v[h:]+v[:h])
    default: // i isn't one of the types above 
    }


json包使用`map[string]interface{}`和`[]interface{}`来存任意的JSON对象或者数组, 

    - bool 对应JSON的布尔值
    - float64 对应JSON的数字
    - string 对应JSON的string
    - nil 对应JSON的null
    

## Decoding arbitrary data ##

    b := []byte(`{"Name":"Wednesday","Age":6,"Parents":["Gomez","Morticia"]}`)

如果提前不知道它的具体结构, 可以定义一个`interface{}`,

    var f interface{}
    err := json.Unmarshal(b, &f)

调用Unmarshal后, f的值为(不能通过map的方式来访问它):

    f = map[string]interface{}{
            "Name": "Wednesday",
            "Age":  6,
            "Parents": []interface{}{
                "Gomez",
                "Morticia",
            },
   }

要访问这个数据, 需要使用类型断言, 然后range遍历它:

    m := f.(map[string]interface{})

使用类型断言来访问底层的数据：

    for k, v := range m {
        switch vv := v.(type) {
        case string:
            fmt.Println(k, "is string", vv)
        case int:
            fmt.Println(k, "is int", vv)
        case []interface{}:
            fmt.Println(k, "is an array:")
            for i, u := range vv {
                fmt.Println(i, u)
            }
        default:
            fmt.Println(k, "is of a type I don't know how to handle")
        }
    }
    

## Reference Types ##

    b := []byte(`{"Name":"Wednesday","Age":6,"Parents":["Gomez","Morticia"]}`)

    type FamilyMember struct {
        Name    string
        Age     int
        Parents []string
    }

    var m FamilyMember
    err := json.Unmarshal(b, &m)

上面这个例子也可以正确地反序列化`b`，`var m FamilyMember`定义了一个FamliyMember结构体实例, 但是，此时`Parents`值是slice nil  , Go会创建一个空的slice, 以免挂掉.
  

再看一个例子：

    type Foo struct {
        Bar *Bar
    }

如果JSON对象包括Bar字段, Unmarshal会分配一个新的Bar, 并且给它初始化, 如果没有, 则为nil

demo如下:

    package main

    import "encoding/json"
    import "fmt"

    type Test struct {
        Demo string
    }

    type Response struct {
        Page   int
        Fruits []string
        T   *Test
    }

    func main() {
        byt := []byte(`{"Page":6, "fruits":["a","b"], "T": {"Demo": "hello world"}}`)

        var t Response
        json.Unmarshal(byt, &t)

        fmt.Println(t.Page) // 6
        fmt.Println(t.Fruits) // [a b]
        fmt.Println(t.T) // &{hello world}
    }
    

## Streaming Encoders and Decoders ##

json包提供了Decoder和Encode类型, 支持常见的读写JSON数据流操作. `NewDecoder`和`NewEncoder`函数对`io.Reader`和`io.Writer`做了包装：

    func NewDecoder(r io.Reader) *Decoder
    func NewEncoder(w io.Writer) *Encoder

下面这个示例, 从标准输入读JSON对象, 剔除除了Name外的所有字段 并写到标准输出

    package main

    import (
        "encoding/json"
        "log"
        "os"
    )

    func main() {
        dec := json.NewDecoder(os.Stdin)
        enc := json.NewEncoder(os.Stdout)
        for {
            var v map[string]interface{}
            if err := dec.Decode(&v); err != nil {
                log.Println(err)
                return
            }
            for k := range v {
                if k != "Name" {
                    delete(v, k)
                }
            }
            if err := enc.Encode(&v); err != nil {
                log.Println(err)
            }
        }
    }
    

## 注意事项 ##

- 只有可以被编码成JSON的数据结构才可以被正确地序列化
- JSON对象的key只能是string类型,如果map类型想编码成JSON数据, 那么其类型必须是`map[string]T`, T是任意的json包支持的Go数据类型
- Channel, complex还有函数类型不能被编码成JSON
- 能形成环路的数据结构不能编码成JSON, 它们会导致Marshal函数陷入死循环
- 指针类型被编码成它们所指向的数据, 如果point是nil, 则返回null
- 结构体中, 只有可被导出的字段才可以被正确地序列化以及反序列化
