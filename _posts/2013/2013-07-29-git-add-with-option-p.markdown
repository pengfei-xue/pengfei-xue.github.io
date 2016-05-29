---
layout: post
title: 'git add with option p'
date: 2013-07-29
author: Pengfei.X
version: 0.1
categories: [others, ]
excerpt: 我有一个文件  里面有多个类 或者多个不怎么相关的函数 有一天 我为了完成两个测试用例 也许是解决QA丢过来的两个bug 我对代码中的几段代码进行了修改 在准备提交的时候 灵光闪现 两次递交 两份钱呐 另外 之后万一 递交的commit里面有bug 写的commit message又很一般 类似 bug fix 这就等着坑爹吧 不好找哪个commit对应了哪个bug
---

我有一个文件  里面有多个类 或者多个不怎么相关的函数 有一天 我为了完成两个测试用例 也许是解决QA丢过来的两个bug 我对代码中的几段代码进行了修改 在准备提交的时候 灵光闪现 两次递交 两份钱呐 另外 之后万一 递交的commit里面有bug 写的commit message又很一般 类似 bug fix 这就等着坑爹吧 不好找哪个commit对应了哪个bug


其实GIT里面有 把本地修改的文件放到stage时 有一个很有用的选项 就是 `git add -p` 看下面这段 在函数cprofile_main 和 main函数中解决了两个BUG 在递交的时候 我想分两次commit 于是使用 `-p` 选项:
    
    git add -p benchmark.py
        diff --git a/benchmark.py b/benchmark.py
        index 16b2fd4..d2aca94 100644
        --- a/benchmark.py
        +++ b/benchmark.py
        @@ -4,6 +4,7 @@ import timeit
        
        
         def cprofile_main():
             +    ''' 修改了这段代码 '''
             from pymongo import Connection
             connection = Connection()
             connection.drop_database('timeit_test')

**Stage this hunk [y,n,q,a,d,/,j,J,g,e,?]?n**

上面这个提示告诉你 是否把这段修改加到stage里去 一般用到的选项就是y和n 也就是所谓的Yes和No了 这段代码我们选择y 之后有一段修改选择为N 之后在看工作区这个文件的状态

    @@ -23,6 +24,7 @@ def cprofile_main():


    def main():
        +    # hack 了这个main函数
         """
         0.4 Performance Figures ...

**Stage this hunk [y,n,q,a,d,/,K,j,J,g,e,?]? y**


查看文件status: 

    # On branch master
    # Changes to be committed:
    #   (use "git reset HEAD <file>..." to unstage)
    #
    #   modified:   benchmark.py
    #
    # Changes not staged for commit:
    #   (use "git add <file>..." to update what will be committed)
    #   (use "git checkout -- <file>..." to discard changes in working directory)
    #
    #   modified:   benchmark.py
    #


更多的可以参考[Interactive Staging](http://git-scm.com/book/en/Git-Tools-Interactive-Staging)
