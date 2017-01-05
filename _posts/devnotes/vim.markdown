---
layout: post
title: 'vim note'
date: 2017-01-05
author: Pengfei.X
version: 0.1
categories: [devnotes, ]
---


## vim linux下打开中文乱码

编辑~/.vimrc文件，加上如下几行：

   set fileencodings=utf-8,ucs-bom,gb18030,gbk,gb2312,cp936
   set termencoding=utf-8
   set encoding=utf-8