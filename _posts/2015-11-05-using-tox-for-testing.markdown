---
layout: post
title: 'using tox for testing'
date: 2015-11-05
author: Pengfei.X
version: 0.1
categories: [python, ]
excerpt: using tox for testing
---

随便翻阅了一下`python 高手之路`，真好最近也真在做一些持续继承的准备工作，看到`tox`
的介绍感觉很棒，亲自试了一下，还是挺好用的，但是，过程中遇到几个坑，记录一下吧！

首先，我们的项目没有`setup.py`文件，但是`tox`默认是去找这个文件的，所以需要在设置
中增加以下这行:

	[tox]
	skipsdist = True

其次，也不是`tox`的问题，是`pip`跟`jpushv3.0.1`闹腾的，`pip`在安装的时候，不会根据
`requirements.txt`中的次序逐个安装，而是自己去找相关的依赖，恰好这个`jpush`版本有
问题，`jpush v3.0.1`依赖了`requests`，写在依赖文件中，但是安装的时候就会报错，无法
找到`requests`，彻底奔溃，解决办法:

	[testenv]
	deps =
		requests==2.5.3

	commands=
		pip install -r {toxinidir}/requirements.txt --index http://pypi.douban.com/simple
		python manage.py test tests

现在环境准备的时候安装`requests`，之后在测试的命令中，安装依赖。

最后，也是最坑爹的，自己的依赖有一些是公司内部的repo，而用`tox`创建新的环境后，一些
环境变量并没有带入到测试的虚拟环境中，导致，找不到`ssh-key`，然后安装就悲剧，解决
办法:

	[testenv]
	passenv = SSH_AUTH_SOCK

最后，完整的`tox.ini`文件内容如下:
	
	[tox]
	envlist = py27

	indexserver =
		default = http://pypi.douban.com/simple

	skipsdist = True

	[testenv]
	envdir = {toxinidir}/.env

	passenv = SSH_AUTH_SOCK

	deps =
		requests==2.5.3

	commands=
		pip install -r {toxinidir}/requirements.txt --index http://pypi.douban.com/simple
		python manage.py test tests	

## references
1. [using tox django projects](http://blog.schwuk.com/2014/03/19/using-tox-django-projects/)
2. [tox, pip, git, and ssh-agent](http://lists.idyll.org/pipermail/testing-in-python/2015-May/006427.html)
3. [Tox tricks and patterns](http://blog.ionelmc.ro/2015/04/14/tox-tricks-and-patterns/)
