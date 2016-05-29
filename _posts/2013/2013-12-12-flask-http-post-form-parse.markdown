---
layout: post
title: 'flask http post form parse'
date: 2013-12-12
author: Pengfei.X
version: 0.1
categories: [python, ]
excerpt: 和同事联调一个web service，他那边使用HTTP POST来检验验证码是否有效，结果他说每次都返回400，我看了日志，发现我没有取到POST的内容。
---


和同事联调一个web service，他那边使用HTTP POST来检验验证码是否有效，结果他说每次都返回400，我看了日志，发现我没有取到POST的内容。


### 先看一段代码 app.py ###


    # -*- coding: utf8 -*-

    from flask import Flask, request, jsonify


    app = Flask(__name__)


    @app.route('/index', methods=['POST', ])
    def index():
        uid = request.form.get('uid', None)
        code = request.form.get('code', None)

        return jsonify(uid=uid, code=code)

    if __name__ == '__main__':
        app.run(debug=True, use_reloader=True)


运行上述代码，使用`curl`来测试一下，看看是否有数据返回：

    curl -X POST http://127.0.0.1:5000/index --data "uid=12x31&code=x164d"

结果是有数据，返回正常:

    {
      "code": "x164d", 
      "uid": "12x31"
    }

于是乎，我告诉他跑这个列子试试，他说可以通过测试，但是，自己写的程序就是没有正确
返回，尝试抓了一下包，两个请求的HTTP头部有差异，`curl`发起的请求有一个HTTP Header
    
     'Content-Type': 'application/x-www-form-urlencoded'

其它都没有什么不同，怀疑是Flask本身再处理的时候，对POST请求数据解析的时候出的问题。



### 跟踪代码 ###

`app.run` 执行后，就可以接受HTTP请求，通过`werkzeug.WSGIRequestHandler`调用`app.__call__`,
Flask会初始化[AppContext](http://flask.pocoo.org/docs/appcontext/)和
[RequestContext](http://flask.pocoo.org/docs/reqcontext/#request-context)，查看
`flask.ctx.RequestContext.__init__`函数，可以看出，`RequestContext`创建了新的`request`
对象:

    def __init__(self, app, environ, request=None):                                
        self.app = app                                                             
        if request is None:                                                        
            request = app.request_class(environ)                                   
        self.request = request                                                     
        self.url_adapter = app.create_url_adapter(self.request)                    
        self.flashes = None                                                        
        self.session = None                                                        

`app.requst_class`定义可以在`flask.app`中看到：

    from .wrappers import Request, Response

    class Flask(_PackageBoundObject):
        ...
        ...
        request_class = Request

再跟踪`wrappers.Reqeust`就可以查看到`request.form`的解析过程。


### Finally the culprit: werkzeug.formparser.py  ###

    def get_parse_func(self, mimetype, options):
        return self.parse_functions.get(mimetype)
                                                                                
    #: mapping of mimetypes to parsing functions                                   
    parse_functions = {                                                            
        'multipart/form-data':                  _parse_multipart,                  
        'application/x-www-form-urlencoded':    _parse_urlencoded,                 
        'application/x-url-encoded':            _parse_urlencoded                  
    }

以上代码可以看出，只有HTTP Header中包含`multipart/form-data, application/x-www-form-urlencoded, application/x-url-encoded`才会正确地解析HTTP POST form数据。


The End
