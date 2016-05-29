---
layout: post
title: 'use line profile with django'
date: 2015-11-09
author: Pengfei.X
version: 0.1
categories: [python, ]
---

之前只用过`linesman`用来给`flask`做profile，现在的项目在用`django`, 另外就是，最近的一
次线上日志发现某个请求的响应时间特别长，平均下来得12-13秒的样子，不管是访问量大小，这样的
响应时间是不可接受的。创业公司，难免任务多，时间短，对于性能方面的测试没有多关注，虽然有段
时间集中做了一些优化，从`sql`到加页面缓存，搜索迁移到`elasticsearch`等，但是，之后的一些
迭代中，就没有再特别关注。

在上班路上看了几篇关于`python`性能优化和`profile`的文章，有一些启发，于是乎，一不做二不休，
在项目中加如了`profile`功能。开始想用`python`自带`cprofile`来完成，但是对于需要对某个特定
函数中，每一行做`profile`就有些力不从心了。google的时候发现了一个gist，直接拿来用来一下：

    class ProfileMiddleware(object):
        def __init__(self):
            if not settings.DEBUG:
                raise MiddlewareNotUsed()
            self.profiler = None

        def process_view(self, request, callback, callback_args, callback_kwargs):
            if settings.DEBUG and ('profile' in request.GET
                                or 'profilebin' in request.GET):
                self.profiler = cProfile.Profile()
                args = (request,) + callback_args
                return self.profiler.runcall(callback, *args, **callback_kwargs)

        def process_response(self, request, response):
            if settings.DEBUG:
                if 'profile' in request.GET:
                    self.profiler.create_stats()
                    out = StringIO()
                    stats = pstats.Stats(self.profiler, stream=out)
                    stats.sort_stats('time').print_stats(.2)
                    response.content = out.getvalue()
                    response['Content-type'] = 'text/plain'
                if 'profilebin' in request.GET:
                    self.profiler.create_stats()
                    response.content = marshal.dumps(self.profiler.stats)
                    filename = request.path.strip('/').replace('/','_') + '.pstat'
                    response['Content-Disposition'] = \
                        'attachment; filename=%s' % (filename,)
                    response['Content-type'] = 'application/octet-stream'
            return response

简单来讲就是打印出`cprofile`的结果。

看了输出后，感觉结果完全没用，太多第三方相关的代码列在了结果中，而真正的业务代码却没有被`profile`
到。继续google后，发现了`line-profiler`，这个库可以对需要做`profile`的代码的每一行做检查。
下面是主要的代码实现：

    def dispatch(self, request, *args, **kwargs):
        self._rpc_client = getattr(request, 'rpc', settings.RPC_INVOKER)
        self._request = request

        if settings.DEBUG and '_profile' in request.GET:
            profiler = LineProfiler()
            handler = getattr(self, request.method.lower(), self.http_method_not_allowed)
            profiler.add_function(handler)

            profiler.enable()
            response = super(Baseview, self).dispatch(request, *args, **kwargs)
            profiler.disable()

            out = StringIO()
            profiler.print_stats(stream=out)
            response.content = out.getvalue()
            response['Content-type'] = 'text/plain'

        else:
            response = super(Baseview, self).dispatch(request, *args, **kwargs)

        return response

输出结果就是我想要的，不错：

    Timer unit: 1e-06 s

    Total time: 8.63884 s
    File: index.py
    Function: get at line 291

    Line #      Hits         Time  Per Hit   % Time  Line Contents
    ==============================================================
       291                                               def get(self, request, *args, **kwargs):
       292         1           10     10.0      0.0          request.logger.app(nothing=True)
       293                                           
       294         1           44     44.0      0.0          start_num = int(request.GET.get("start_num", "0"))
       295         1           13     13.0      0.0          tabtype = int(request.GET.get("tabtype","0"))
       296         1            2      2.0      0.0          count = 10
       297         1            2      2.0      0.0          error = 0  
       298         1            2      2.0      0.0          data = {}
       299         1          135    135.0      0.0          pos_0 = slide_adapter_future(position=0)
       300         1      8355318 8355318.0     96.7          data['slides'] = pos_0.unwrap()
       301         1           88     88.0      0.0          map(lambda x: _prune_slide_keys(x), chain(data['slides'],))


## references
1. [line_profiler source code](https://github.com/rkern/line_profiler/blob/master/line_profiler.py)
2. [django cprofile middleware](https://gist.github.com/kesor/1229681)
3. [检测Python程序执行效率及内存和CPU使用的7种方法](http://python.jobbole.com/80754/)
4. [使用cProfile分析Python程序性能](http://xianglong.me/article/analysis-python-application-performance-using-cProfile/)
5. [The Python Profilers](https://docs.python.org/2/library/profile.html)
