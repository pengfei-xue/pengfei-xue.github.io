---
layout: post
title: 'threading bug in python2'
date: 2014-03-22
author: Pengfei.X
version: 0.1
categories: [python, ]
excerpt: 调试代码发现threading.Lock.lock()抛异常
---

使用的Python版本<b>2.7.3</b>, Gunicorn <b>(version 18.0)</b>


最初没有发现为什么出现这个问题，但是，程序运行似乎有些问题。
我的程序里面使用Lock来保证同一个文件夹只被一个进程使用，
可实际结果却不是。查看日志，调试代码发现threading.Lock.lock()抛异常:

    Exception AttributeError: AttributeError("'_DummyThread' object has no attribute '_Thread__block'",) in <module 'threading'

继续看threading模块，在哪里会创建_DummyThread呢？看下面代码：

    def currentThread():
       try:
           return _active[_get_ident()]
       except KeyError:
           ##print "current_thread(): no current thread for", _get_ident()
           return _DummyThread()

如果调用到了threading.currentThread就会返回一个DummyThread对象，并且DummyThread在初始化的时候，
还把__block给删除了：

    class _DummyThread(Thread):
        def __init__(self):
            Thread.__init__(self, name=_newname("Dummy-%d"))
  
            # Thread.__block consumes an OS-level locking primitive, which
            # can never be used by a _DummyThread.  Since a _DummyThread
            # instance is immortal, that's bad, so release this resource.
            del self._Thread__block

ok, 所以在对_DummyThread对象调用__block时，就会产生Exception。再看_after_fork函数，该函数在创建线程后，
执行一些清理工作，其中，有个地方调用了`thread._Thread__stop()`, 当这个thread正好是_DummyThread的时候，
就会出错。


找了一些资料，threading库有个bug，相关的patch如下(也是解决方法)：

    changeset:   76422:e3ea462cb181                                                 
    parent:      76419:293180d199f2                                                 
    parent:      76421:41c64c700e1e                                                 
    user:        Antoine Pitrou <solipsis@pitrou.net>                               
    date:        Fri Apr 20 00:05:17 2012 +0200                                     
    summary:     Issue #14308: Fix an exception when a dummy thread is in the threading module's active list after a fork().
                                                                                    
    diff -r 293180d199f2 -r e3ea462cb181 Lib/threading.py                           
    --- a/Lib/threading.py  Thu Apr 19 18:21:04 2012 +0200                          
    +++ b/Lib/threading.py  Fri Apr 20 00:05:17 2012 +0200                                                              
    @@ -871,6 +871,9 @@                                                             
             with _active_limbo_lock:                                               
                 _active[self._ident] = self                                        
                                                                                    
    +    def _stop(self):                                                           
    +        pass                                                                   
    +                                                                               
         def join(self, timeout=None):                                              
             assert False, "cannot join a dummy thread"                             
                                                                                    
当然，最好的办法还是升级Python到2.7.4+

## 参考 ##

1. [Message160297](http://bugs.python.org/issue14308)
2. [Understand python threading bug](http://stackoverflow.com/questions/13193278/understand-python-threading-bug)
3. [Night Of The Living Thread](http://emptysqua.re/blog/night-of-the-living-thread/)
4. [Fix an exception when a "dummy" thread is in the threading module's active list after a fork()](http://hg.python.org/cpython/rev/41c64c700e1e)
