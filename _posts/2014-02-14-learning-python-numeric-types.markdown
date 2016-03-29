---
layout: post
title: 'learning python numeric types'
date: 2014-02-14
author: Pengfei.X
version: 0.1
categories: [python, ]
excerpt: 一些关于Python numeric类型的读书笔记
---

最近有同事在看Python Cookbook， 问了几个问题，很多内容需要知道Learning Python
中提及的小细节。回过头来看这本书，获得的东西肯定是和当初入门时不一样，看了前面
几章通俗介绍后，一些平时不注意到的细节可以记录一下，于是，有了这篇文章。


### 平时不注意的事项

1. built-in function `bin`

    之前没有注意到这个内建函数，作用其实很简单，就是把整数转换为二进制格式。

        In [10]: bin(20140214)
        Out[10]: '0b1001100110101000010110110'


2. [start:end:step]

    我们用的最多的是只有start和end，没有step，至少我自己是不常用第三个参数。
    step指步进值，每隔step取一个数，比如，有一个list

        a = [0,1,2,3,4,5,6,7,8]

    我们需要每隔2取值:

        In [16]: a[::2]
        Out[16]: [0, 2, 4, 6, 8]

    亦或者，我们需要把list内容掉个个, step可以为负数，这样的话，从list的尾部开始
    取值:

        In [17]: a[::-1]
        Out[17]: [8, 7, 6, 5, 4, 3, 2, 1, 0]


3. set内的元素必须是immutable
    
        >>> s = set([1,2])
        >>> s.add([1,2])
        Traceback (most recent call last):
          File "<stdin>", line 1, in <module>
        TypeError: unhashable type: 'list'


### py2 vs. py3

1. long and normal int type(merged in py3)

    Py2中，整数有普通整数和长整数(尾部加了l或者L)，但是，在Py3中，已经把这两部分
    合并，只有整数，不区分长短：

        Python 2.7.5 (default, Nov 12 2013, 16:45:54) 
        [GCC 4.8.2 20131017 (Red Hat 4.8.2-1)] on linux2
        >>> 10L
        10L

        Python 3.3.2 (default, Nov  7 2013, 10:01:05) 
        [GCC 4.8.1 20130814 (Red Hat 4.8.1-6)] on linux
        >>> 10L
          File "<stdin>", line 1
          10L
            ^
        SyntaxError: invalid syntax
        >>> 10
        10


2. octal in py2 vs. py3

    八进制值也有做了一些改动，Py2中，八进制可以通过0或者0o来表示：

        >>> 071
        57
        >>> 0o71
        57

    但是，在Py3中，只可以以0o开头来表示：
        
        >>> 071
          File "<stdin>", line 1
          071
            ^
        SyntaxError: invalid token
        >>> 0o71
        57

    (0o为零，字母欧，大小写都可以)


3. repr vs. str

    Python中表示，或者说显示一个对象有两种形式，一种是所谓的code，另外一种是字符串
    形式，后者对人比较友好，

        class T(object):
            def __str__(self):
                return "T"
        
        t = T()
        >>> t
        <__main__.A object at 0x7ff06d3b4550>
        >>> print t
        A

    上面代码中，`print t`是字符串形式，而直接`t`显示的就是code形式。

    Py2中可以使用`repr`或者`var`的方式来显示对象的code形式，但是，在Py3中已经把
    第二种方法移除：

        Python 2.7.5 (default, Nov 12 2013, 16:45:54) 
        [GCC 4.8.2 20131017 (Red Hat 4.8.2-1)] on linux2
        >>> `a`
        '<__main__.A object at 0x7ff06d3b4550>'
        >>> repr(a)
        '<__main__.A object at 0x7ff06d3b4550>'

        Python 3.3.2 (default, Nov  7 2013, 10:01:05) 
        [GCC 4.8.1 20130814 (Red Hat 4.8.1-6)] on linux
        >>> `a`
          File "<stdin>", line 1
          `a`
          ^
        SyntaxError: invalid syntax
        >>> repr(a)
        '<__main__.A object at 0x7ff06d3b4550>'


4. division除法

    除法操作在Py3中为经典除法，而在Py2中是地板除, Py3中如果需要地板除，则使用`//`：
        
        Python 2.7.5 (default, Nov 12 2013, 16:45:54) 
        [GCC 4.8.2 20131017 (Red Hat 4.8.2-1)] on linux2
        >>> 2 / 3
        0
        >>> -5 / 2
        -3

        Python 3.3.2 (default, Nov  7 2013, 10:01:05) 
        [GCC 4.8.1 20130814 (Red Hat 4.8.1-6)] on linux
        >>> 2 / 3
        0.6666666666666666
        >>> 2 // 3
        0
        >>> -5 // 2
        -3
