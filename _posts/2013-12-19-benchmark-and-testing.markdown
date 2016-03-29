---
layout: post
title: 'benchmark and testing'
date: 2013-12-19
author: Pengfei.X
version: 0.1
categories: [golang, ]
excerpt: benchmark function using package testing
---


最近写了一个图片存储的服务，比较原始，需要直接读写磁盘，寻找磁盘上文件是否存在，
是否修改过，于是，像测试一下读写文件的性能。

Linux本身会对文件进行cache，我们的图片都是一些小图片，20-30KB左右，所以，读一次之后
借助系统提供的cache功能，效率上可以提升很多, 内存足够的情况下不需要对磁盘再次读写。
测试代码(截取)如下：

    package main

    import (
        "io/ioutil"
        "testing"
        "fmt"
    )

    func ReadFile(p string) ([]byte, error) {
        content, err := ioutil.ReadFile(p)
        if err != nil {
            return nil, err
        }

        return content, nil
    }

    func Benchmark(b *testing.B) {
        sum := 0

        for i := 0; i < b.N; i++ {
            content, _ := ReadFile("/tmp/t_test.go")
            sum += len(content)
        }

        fmt.Println(sum)
    }
    

运行测试：

    go test -file t_test.go -bench=".*"

    // output
    testing: warning: no tests to run
    PASS
    Benchmark-4
    384
    38400
    3840000
    192000000

    500000          6148 ns/op
    ok    _/tmp   3.146s

    
## 参考 ##

1. [difference between buffer and page cache](http://www.quora.com/Linux-Kernel/What-is-the-major-difference-between-the-buffer-cache-and-the-page-cache)
2. [Experiments and fun with the Linux disk cache](http://www.linuxatemyram.com/play.html)
