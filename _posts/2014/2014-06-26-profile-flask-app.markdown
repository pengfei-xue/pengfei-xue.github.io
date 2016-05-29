---
layout: post
title: 'profile flask app'
date: 2014-06-26
author: Pengfei.X
version: 0.1
categories: [python, ]
excerpt: profile flask app
---

一直在找python web应用profile的工具 我们使用了flask 昨天使用werkzeug的profile中间件 但是生成的结果日志没办法图像化 看起来很很费力 最终选择放弃

Google许久 总算找到一个比较不错的profile工具 可以直接通过web界面看到profile的结果 并且可以得到图像化调用栈 该库的地址 http://pythonhosted.org/linesman/narr/getting_started.html

使用方法：


from linesman.middleware import make_linesman_middleware

app.wsgi_app = make_linesman_middleware(
    app.wsgi_app, 
    profiler_path="/__profiler")
上述代码中的profiler_path是可以通过web访问的uri 完整url为：http://yourtestserver:port/__profiler

结果截图：

<img src="http://static.oschina.net/uploads/space/2013/0425/144632_GCCV_811778.png"/>
