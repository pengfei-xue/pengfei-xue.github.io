---
layout: post
title: 'git clean up working space'
date: 2013-12-31
author: Pengfei.X
version: 0.1
categories: [others, ]
excerpt: 经常吧工作目录搞的乱七八糟的，需要时不时地清理一下
---

很多时候需要为自己的工作目录清理一下，比如，常见的有恢复当前版本
到HEAD，保存当前的工作，check out出来另一个分支做开发，下面介绍几个我自己常用
的方法。
   

### git stash ###

保存当前的工作，checkout出来其它的分支进行开发。比如，我在更新一个Readme文件，
而同时，我又有一个新的任务进来，需要checkout另外一个分支。那么这个时候，我就
需要保存当前的工作，一个很好用的方法就是`git stash`

    git st
    # On branch master
    # Changes not staged for commit:
    #   (use "git add <file>..." to update what will be committed)
    #   (use "git checkout -- <file>..." to discard changes in working directory)
    #
    #   modified:   README
    #
    no changes added to commit (use "git add" and/or "git commit -a")

    
    git stash
    # Saved working directory and index state WIP on master: afd3c45 Rewrapped lines
    # HEAD is now at afd3c45 Rewrapped lines

    git stash show 
    #  README | 2 +-
    #   1 file changed, 1 insertion(+), 1 deletion(-)

    git stash list 
    #   stash@{0}: WIP on master: afd3c45 Rewrapped lines

我们把当前的工作进度保存到了stash中，然后，可以使用`git stash list`来查看当前的stash
状态，当我们做完其它的事情，需要恢复之前的工作，那么可以使用`git stash pop`或者`git stash apply stash@{0}。

    git st
    # On branch master
    nothing to commit, working directory clean

    git stash pop 
    # On branch master
    # Changes not staged for commit:
    #   (use "git add <file>..." to update what will be committed)
    #   (use "git checkout -- <file>..." to discard changes in working directory)
    #
    #   modified:   README
    #
    no changes added to commit (use "git add" and/or "git commit -a")
    Dropped refs/stash@{0} (9ad1432e3e829852a82bb43c40f279fc441e657e)

更多的stash用法，可以使用`git help stash`或者去baidu，google一下。
    

### 删除当前工作目录中 不在版本控制中的文件 ####

很多时候，我们会做个build，然后呢，工作目录中多了一堆不需要的文件，文件夹，看着特别
别扭。也许，你会说，没关系啊，我早就把这些生成的中间文件，文件夹写到`gitignore`里面去
了，这些多余的货色，不会影响到我的工作，也不会把仓库搞乱啊。恩，是有道理。但是，就像
眼里多了一个沙子一样，总是想要揉掉它的。`git clean`就可以用来做这件事情，干净利落，比one
更加赞。

    git st
    # On branch master
    # Untracked files:
    #   (use "git add <file>..." to include in what will be committed)
    #
    #   abc.txt
    #   test/
    nothing added to commit but untracked files present (use "git add" to track)

    git clean -f -d
    # Removing abc.txt
    # Removing test/

    git st
    # On branch master
    nothing to commit, working directory clean

啊，她还是原来的她，不乱了。
    

### 清理远程分支 ###

如果在github上贡献了自己的代码，并且在github上删除了一个分支，那么，在本地，还是能看到
这个分支存在。你这个时候去push会失败，我们需要prune一下远程分，把不存的分支信息从本地删除。
可以通过`git remote prune <remote>`来实现。

    git branch -a
    # * develop
    #   master
    #   remotes/origin/HEAD -> origin/master
    #   remotes/origin/develop
    #   remotes/origin/gh-pages
    #   remotes/origin/master

    git push origin :develop
    # error: unable to delete 'develop': remote ref does not exist
    # error: failed to push some refs to 'git@github.com:pengfei-xue/beego.git'

    git remote prune origin 
    # Pruning origin
    # URL: git@github.com:pengfei-xue/beego.git
    #  * [pruned] origin/develop
