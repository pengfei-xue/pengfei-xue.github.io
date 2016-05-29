---
layout: post
title: 'vim jedi python plugin'
date: 2013-11-27
author: Pengfei.X
version: 0.1
categories: [others, ]
excerpt: 使用过的唯一VIM的插件 就是Jedi-VIM，使用介绍和其相关特性可以在项目的页面找到，这边主要把安装的步骤写一下，存档。
---


使用过的唯一VIM的插件 就是Jedi-VIM，[项目地址][1]，使用介绍和其相关特性可以在项目的页面找到，这边主要把安装的步骤写一下，存档。

1. 安装pathogen.vim
  
        mkdir -p ~/.vim/autoload ~/.vim/bundle; \
        curl -Sso ~/.vim/autoload/pathogen.vim \
        https://raw.github.com/tpope/vim-pathogen/master/autoload/pathogen.vim

在～/.vimrc中添加  `execute pathogen#infect()`



2. 安装jedi-vim

        cd ~/.vim/bundle
        git clone https://github.com/davidhalter/jedi-vim
        cd jedi-vim
        git submodule update --init

o了，但是直接装上后，编辑Py文件的时候会自动缩进，而且是用tab来进行缩进，tab的长度为8个whitespace，我的习惯是用4个whitespace缩进，可以在.vimrc中添加如下配置：

        set tabstop=4
        set expandtab
        set softtabstop=4  # 主要是这两个设置 
        set shiftwidth=4  # 主要是这两个设置 
        filetype plugin indent off
        set noautoindent
        set nosmartindent
        set nocindent
        autocmd FileType python setlocal completeopt-=preview


[1]: https://github.com/davidhalter/jedi-vim
