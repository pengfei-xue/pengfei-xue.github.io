---
layout: post
title: 'security issues'
date: 2015-12-04
author: Pengfei.X
version: 0.1
categories: [others, ]
---


公司有同事的邮箱账号登陆异常，黑客在搞我们，于是乎，请了乌云白帽子，给我们来了一次安全性测试，
整个过程持续了十来天，第一天开始搞的时候，异常兴奋，从来没有经历过这样的事情，想了解一下黑客
们是怎么做入侵的。提前说一句后话，如果没有乌云的后台，我们还能知道自己被攻击了么？可能要过很久
出现了一些事故后，才能知道，这一点蛮惨的。

以下是乌云后台相关漏洞的截图：

<img src="/images/common_security_holes.png" style="width: 640px"/>

都比较常见，主要是五点：
- 开发过程中，没有对用户产生内容做过滤，导致出现xss
- 管理不完善，存在大量的后台弱口令
- 开发测试环境没有做隔离
- 员工安全性意识差
- 没有对第三方工具做安全评估

截图中，redis访问，漏洞-数据库，Django开启了debug泄漏敏感信息，这几项属于开发测试环境没有隔离，
外网可以直接访问开发测试环境，一旦出现错误页面，我们的开发框架就会把敏感信息泄漏出去，比如，数据
库连接信息，万一开发测试的数据库的用户密码跟生产环境一致，那就只能呵呵了。redis访问问题，是我们
部分同事，再使用配置redis的时候，绑定的IP是`0.0.0.0`，结果导致了端口暴露到了外网，外网可以直接
连接上redis，读取相关数据。

运营后台弱口令，某后台限制不严格，大量账号弱口令这些属于管理上的问题，运营同事在设置密码时，
没有使用复杂的密码，而是一些`123456`，`19880120`，同时，我们运营后台没有对密码试错做处理，给
暴力破解留下了空间。密码设置上，不要出现跟自己身份信息有关的内容，比如，自己的电话号码，生日，姓名
拼音，纯数字或者纯密码，一旦被顶上，用一些简单的社工方法就可以破解。所以，在设置密码的时候，推荐
使用密码生成器，或者密码中加入数字，字母，符号，并且长度不能少于6位。

xss，ios APP存储性xss，这些漏洞属于没有对用户产生的内容做过滤，导致用户可以在发帖子，评论时，
注入`js`代码，当加载页面的时候，直接执行了注入的`js`代码，导致用户信息泄漏。防御xss的常见方式是，
对所有用户的输入做转义，有很多现成的库提供了这方面的功能，但是，我们自己在实际开发过程中，缺忽视了
这一点，要命。另外，在设置`cookie`的时候，需要设置`httponly`属性，加载`js`的时候，可以避免
`cookie`被带过去。对于XSS的常见形式和更多的防御方式，可以参见`白帽子讲安全`。

我们用的项目管理工具，在这次测试中，不幸中招，导致员工信息泄漏，给社工工作做了铺垫，在使用第三方
工具的时候，我们需要对其安全性做评估，防止一些后门，留下安全隐患。

事后，我们对开发测试环境，运营后台访问都做了限制，只有内网才可以访问，并且设置了vpn，让不在公司的
同事，通过vpn访问这些环境，同时，通知了所有同事，修改自己的密码，杜绝后台弱口令。邮箱管理方面，我们
用的是QQ的企业邮箱，让所有同事，都绑定了微信登陆提示，一旦出现登陆失败，或者检测到攻击时，就会给相关
人发微信提醒。在加载页面内容时，统一做了转义，防止注入的`js`代码在页面上执行。在vpn使用方面，我们强化
了密码管理，登陆vpn的密码是个人密码加谷歌验证的6位校验码。
