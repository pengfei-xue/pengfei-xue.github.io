---
layout: post
title: 'mac can not access network after connected with vpn'
date: 2013-08-04
author: Pengfei.X
version: 0.1
categories: [others, ]
excerpt: 想在家里看点东西 公司VPN连上后 只能SSH访问 公司网页怎么都打不开 而且有一个症状是 SSH到远程机器后 如果执行`ls -al` 或者是用`vim`打开某个稍微大一些的文件的时候 似乎就挂了 没有反应了
---

想在家里看点东西 公司VPN连上后 只能SSH访问 公司网页怎么都打不开 而且有一个症状是 SSH到远程机器后 如果执行`ls -al` 或者是用`vim`打开某个稍微大一些的文件的时候 似乎就挂了 没有反应了


对比了公司里的VPN配置手册 没啥问题 开始是以为之前改过路由表的问题 但是清除自己加的路由后 还是相同的症状 搞了个半死 最后看到了几篇文章 说是MTU过大 超过网关的MTU 包就会被抛弃 放回的信息就是目的不可达 `ping -s 1500 gateway` 直接就是返回不可达了 如果调小MTU 一些都正常了 唉 基础知识不行害人啊
