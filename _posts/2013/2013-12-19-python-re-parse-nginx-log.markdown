---
layout: post
title: 'python re parse nginx log'
date: 2013-12-19
author: Pengfei.X
version: 0.1
categories: [python, ]
excerpt: Python re应用
---


上午同事问了一个问题，怎么用Python提取下面这个字符串中`[]`内的数据：

     [2013-12-08 21:23:04] [http://localhost:9001/ef-front/v1/a/stores/detail/6] [0:0:0:0:0:0:0:1] [Mozilla/5.0 (Macintosh; Intel Mac OS X 10.9; rv:25.0) Gecko/20100101 Firefox/25.0] [a] [1] [] [null] [null] [null] [null] [null] [null] [0] [0][0]

工作第一年是写的`perl`, 所以，看了这个，肯定是用`re`来解析这个字符串，下面是代码：

    import re

    s = '[2013-12-08 21:23:04] [http://localhost:9001/ef-front/v1/a/stores/detail/6] [0:0:0:0:0:0:0:1] [Mozilla/5.0 (Macintosh; Intel Mac OS X 10.9; rv:25.0) Gecko/20100101 Firefox/25.0] [a] [1] [] [null] [null] [null] [null] [null] [null] [0] [0][0]'

    pattern = re.compile(r'\[(.*?)\]')
    pattern.findall(s)

    
Done
