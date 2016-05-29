---
layout: post
title: 'twelve factor and how we do'
date: 2013-12-05
author: Pengfei.X
version: 0.1
categories: [others, ]
excerpt:  目前为止做过的项目，大多是提供API，做一些基础的服务，前两天看golang的一个blog，发现了一篇好文章，the twelve factor app。 看了三章，发现里面有很多条建议我们一直在使用。
---

目前为止做过的项目，大多是提供API，做一些基础的服务，前两天看golang的一个blog，
发现了一篇好文章，[the twelve factor app](http://12factor.net/)。 看了三章，发现
里面有很多条建议我们一直在使用，这篇文章算是一个小结吧！


## Codebase ##

这个factor建议每个APP就独占一个Repo，部署在开发，测试，预发布和生产环境的代码都
可以追溯到同一个Commit。这中方式有利于APP as a Service的构架，每个APP都有其特定
的职能范围。估计99.99%的APP都是这样做的。

我们内部在使用Gitlab作为版本管理仓库，每个项目都是一个独立的Repo。 当然也有使用
`submodule`方式引入其它项目的，但是，这种情况很少，我知道的只有一个项目。我们小
组自己开发了一个发布系统，可以把项目打包成RPM，然后使用YUM来进行安装，升级等。每
个项目都会由自己独立的RPM SPEC文件，需要发布的时候，提供相应的tag来打包。


## Dependencies ##

管理依赖是个比较麻烦的事情，最近最到的一次事故，就是手痒升级了一个框架，结果里面
的API变了，直接导致应用挂了。

我们自己的APP大多是Python写的，用`pip requirements`来管理依赖，每个项目都必须有
这个文件，申明所有的依赖库。但是，最近比较头疼的是Go项目的依赖管理，没有找到比较
给力的第三方工具来管理Go的依赖。而我们的打包工具是用直接生成Go的可执行文件的，我
现在的做法是，在打包环境建立依赖的Git库，然后手工签出需要的commit，之后打包就可以
保证环境依赖不会出问题了。虽然，可以解决眼前的问题，但能有个给力的工具还是更好一
些。

Go的依赖管理，我觉得应该出一个类似Maven的工具，毕竟有很多相似的地方，比如，依赖
大多是某个Git库，或是其它的一些版本管理的库地址。Maven的资源定位的方式和Go语言
`import`依赖，在语法上有一定的相似度。类似的工具应该会出现吧。


## Config ##

把配置文件从代码中分离出来，带来的好处就是同一份代码，可以根据需要改变它的一些
行为。

我们在开发，测试，生产环境中使用的方式是，通过[Supervisor](http://supervisord.org/)
来管理应用，并且使用环境变量来指定配置文件的地址。例如：

    [program:app]
    command = /usr/local/python/bin/gunicorn server:app -c /web/app/configs/gunicorn.conf
    directory = /web/app
    environment = APPCONFIG ="/etc/app/production.cfg"
    process_name = app


## Backing Services ##

通常我们的APP都会有DB，或者Cache系统的支持，我们可以把这些也看成是一个服务，我们
的代码里不应该和这些服务耦合的太厉害，应该可以通过配置文件，切换不同的支持服务，
比如，MySQL连接，在测试环境和生产环境连接的一般都不是同一个，所以，我们代码里面
不可以把这部分信息写死，应该通过配置文件来达到切换的目的。


## Build, release, run ##

我们自己开发了打包的系统，很简单，就是把代码打包程RPM，RPM包的版本通过tag来设定。
服务器上安装的是CentOS系列的操作系统，所以，结合YUM使用非常方便。之前在配置一条
中已经提到，我们的配置文件都是通过环境变量来指定的，所以只需要把代码打包就可以了。
发布的话，通过`yum upgrade`来进行。

Python项目，打包就是把源码打包成RPM，一般不需要Build过程。运行环境通过supervisor
来管理，可以把APP重启的指令放在RPM的`%post`小节里面，安装升级后，直接重启应用。
例如：

    %post
    /usr/local/python/bin/supervisorctl restart app

Go项目，就需要Build来产生可以执行文件了，这个可以在RPM打包过程中进行。具体的RPM
打包细节请参考[RPM Guide](http://docs.fedoraproject.org/en-US/Fedora_Draft_Documentation/0.1/html/RPM_Guide/)


## Dev/prod parity ##

开发，测试，预发布和正式环境应该尽可能地一致，有很多管理的工具可以实现，比如，使用
Puppet。或者，直接克隆虚拟机来达到这个目的。
