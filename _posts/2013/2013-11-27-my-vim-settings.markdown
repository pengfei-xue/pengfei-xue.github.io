---
layout: post
title: 'my vim setttings'                                     
date: 2013-11-27
author: Pengfei.X
version: 0.1
categories: [others, ]
excerpt: my vim settings
---

    set ruler
    set showcmd
    set tabstop=4
    set expandtab
    set softtabstop=4
    set shiftwidth=4
    set nu
    set hlsearch
    set incsearch
    set foldmethod=marker
    set colorcolumn=80
    set statusline=%<\ %n:%F\ %m%r%y%=%-35.(line:\ %l\ of\ %L,\ col:\ %c%V\ (%P)%)
    set laststatus=2
    set nosi
    set noai

    colorscheme blackdust


    "so that when you preview the new file takes up 80% and the file explorer the other 20%.
    let g:netrw_winsize=20

    syntax on

    set cursorline
    hi clear CursorLine
    hi CursorLine gui=underline
    hi CursorLine ctermfg=NONE ctermbg=Blue

    let g:netrw_altv = 1
    let g:netrw_browse_split = 3
    let g:netrw_keepdir = 0

    map <F2> :mksession! ~/.vim_session <cr> " Quick write session with F2
    map <F3> :source ~/.vim_session <cr>     " And load session with F3

    execute pathogen#infect()
    let g:jedi#completions_command = "<C-Space>"
    filetype plugin indent off
    set noautoindent
    set nosmartindent
    set nocindent
    autocmd FileType python setlocal completeopt-=preview


    function! ResCur()
            if line("'\"") <= line("$")
                    normal! g`"
                    return 1
            endif
    endfunction

    augroup resCur
            autocmd!
            autocmd BufWinEnter * call ResCur()
    augroup END

    au BufNewFile,BufRead *.yml set filetype=xml
