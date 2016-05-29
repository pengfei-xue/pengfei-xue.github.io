---
layout: post
title: 'my tmux setttings'                                     
date: 2013-11-27
author: Pengfei.X
version: 0.1
categories: [others, ]
excerpt: my tmux settings
---

    set -g prefix ^a
    unbind ^b
    bind a send-prefix

    unbind '"'
    bind - splitw -v # 分割成上下两个窗口
    unbind %
    bind | splitw -h # 分割成左右两个窗口

    bind k selectp -U # 选择上窗格
    bind j selectp -D # 选择下窗格
    bind h selectp -L # 选择左窗格
    bind l selectp -R # 选择右窗格

    bind-key y set-window-option synchronize-panes
    bind-key u set-window-option synchronize-panes off

    set-option -g status-left '#[fg=colour240]#S:#I |'
    set-window-option -g window-status-format '#[fg=colour240]#F#[fg=default]#W#[fg=colour240]#F'
    set-window-option -g window-status-current-format '#[fg=colour240]#F#[fg=default]#W#[fg=colour240]#F'
    set-option -g status-right '#[fg=colour240]| %a %b %d %I:%M %p'
    set-option -g status-bg colour234
    set-option -g status-fg colour007
    set-window-option -g window-status-current-fg colour208

    set -g mode-mouse on

    setw -g mode-keys vi
    bind-key -t vi-copy 'v' begin-selection
    bind-key -t vi-copy 'c' copy-selection
    bind x run-shell "tmux show-buffer | xclip -sel clip -i" \; display-message "Copied tmux buffer to system clipboard"


Reference:

[A tmux Crash Course](http://robots.thoughtbot.com/a-tmux-crash-course)
[从 screen 切换到 tmux](http://linuxtoy.org/archives/from-screen-to-tmux.html)
