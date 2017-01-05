---
layout: post
title: 'golang note'
date: 2017-01-05
author: Pengfei.X
version: 0.1
categories: [devnotes, ]
---


## go by example

    https://gobyexample.com/


## Golang: format a string without printing?

    return fmt.Sprintf("at %v, %s", e.When, e.What)

[Golang: format a string without printing?](http://stackoverflow.com/questions/11123865/golang-format-a-string-without-printing)


## How to find a type of a object in Golang?

    package main

    import (
        "fmt"
        "reflect"
    )

    func main() {

        tst := "string"
        tst2 := 10
        tst3 := 1.2

        fmt.Println(reflect.TypeOf(tst))
        fmt.Println(reflect.TypeOf(tst2))
        fmt.Println(reflect.TypeOf(tst3))

    }

    // 或者
    func typeof(v interface{}) string {
        return fmt.Sprintf("%T", v)
    }

    // 或者
    func typeof(v interface{}) string {
        return reflect.TypeOf(v).String()
    }

    // 或者
    func typeof(v interface{}) string {
        switch t := v.(type) {
        case int:
            return "int"
        case float64:
            return "float64"
        //... etc
        default:
            _ = t
            return "unknown"
        }
    }

[How to find a type of a object in Golang?](http://stackoverflow.com/questions/20170275/how-to-find-a-type-of-a-object-in-golang)