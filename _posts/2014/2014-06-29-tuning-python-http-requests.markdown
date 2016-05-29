---
layout: post
title: 'tunning requests, add dns cache'
date: 2014-06-29
author: Pengfei.X
version: 0.1
categories: [python, ]
excerpt: tunning requests add dns cache
---

开发基本告一段落，开始对代码进行一些优化，使用的flask框架，做profile的时候，使用
linesman来记录profile的数据，并展示。linesman比较给力的一个地方是，它的web界面很
好用，可以直观地看到应用在哪里话费的时间比较多，方便定位问题所在。


下图是没有优化前，一个HTTP请求的调用时间，已经call stack:

<img src="/images/before-tunning.png" style="width: 640px"/>

可以看到用黑笔标记的部分，每次发起HTTP请求，都会去解析外部服务的地址，导致请求的
时间变长，影响整体的响应速度。首先，我们把两次HTTP请求合并成一次，这样在一定程度上
加速了应用，减少的时间就是一次HTTP请求的时间，大概在0.2-0.3s之间。

但是，整体的响应速度还是不理想，在0.7-0.9s之间，之后，我们把DNS做了cache，这样，
在调用外部服务时，DNS解析部分就节省了大量的时间，优化后的profile如下：

<img src="/images/after-tunning.png" style="width: 640px"/>

DNS cache原理如下：

    import _socket
    import socket

    import requests


    prv_getaddrinfo = socket.getaddrinfo
    dns_cache = {}
    def new_getaddrinfo(*args):
        try:
            print 'got cached'
            return dns_cache[args]
        except KeyError:
            res = prv_getaddrinfo(*args)
            dns_cache[args] = res
            print 'here'
            return res
    socket.getaddrinfo = new_getaddrinfo
