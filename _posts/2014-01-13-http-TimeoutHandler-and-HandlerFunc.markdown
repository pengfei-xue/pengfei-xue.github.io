---
layout: post
title: 'http timeouthandler and handlerfunc'
date: 2014-01-13
author: Pengfei.X
version: 0.1
categories: [golang, ]
excerpt: 之前看一片Go HTTP middleware的文章，里面介绍了两种写middlware的方法，看完之后，感觉自己对Go还只是入门而已，这货太灵活了。
---


## HandlerFunc ##

Go的`net/http`中有一个Handler接口，定义如下:

    type Handler interface {
        ServeHTTP(ResponseWriter, *Request)
    }

按照定义，我们可以自己创建一个类型，实现Handler接口，这也是我的做法。但在包中还有一个方法可以使用，

    type HandlerFunc func(ResponseWriter, *Request)

可以使用这个类型来包装我们自己的一个函数，这样做可以省去自定义类型的麻烦，直接写一个函数，函数
签名和ServerHTTP一致就可以了。第一次看到这篇文章的时候，没有特别注意这个用法，一带而过来。后然，自己
需要写个middleware，于是，再次把文章读了一遍，再看到这个用法，感觉很新鲜，有点像Python里面的装饰器。

看一下这个HandlerFunc是如何工作的:

    type HandlerFunc func(ResponseWriter, *Request) 

    func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
        f(w, r)
    }

Go中函数也是一种类型，ok，解释结束。


## TimeoutHandler ##

TimeoutHandler的做法比较常见了，定了timeoutHandler类型，该类型对Handler做了封装，

    type timeoutHandler struct {
        handler Handler
        timeout func() <-chan time.Time
        body    string
    }

    func TimeoutHandler(h Handler, dt time.Duration, msg string) Handler {
        f := func() <-chan time.Time {
            return time.After(dt)
        }
        return &timeoutHandler{h, f, msg}
    }


## 参考 ##

1. [Writing HTTP Middleware in Go](http://justinas.org/writing-http-middleware-in-go/)
