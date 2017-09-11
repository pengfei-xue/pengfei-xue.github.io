---
layout: post
title: 'flask & sqlalchemy snippets'
date: 2014-06-18
author: Pengfei.X
version: 0.1
categories: [python, ]
excerpt: flask sqlalchemy的一些snippets
---


### url_for跳转到HTTPS站点

使用url_for生成的地址一版都是HTTP的地址，有的时候需要转到HTTPS的时候可以
使用两外两个参数来让生成的URL跳转到外部的HTTPS链接上：

    import flask
    def patch_url_for():
        url_for_https = functools.partial(flask.url_for, _scheme='https', _external=True)
        setattr(flask, 'url_for', url_for_https)
    patch_url_for()

这里对flask的url_for做了patch，之后用到url_for时，生成的地址都会转到外部的HTTPS上去。


### 生成CSRF TOKEN

我们需要在处理POST请求时，使用csrf来防止伪造的跨站攻击请求，可以在jinja2里注入一个生成
CSRF TOKEN的函数：

    import random
    from flask import session

    def random_str(length=8):
        chars = '01234567890abcdefghijklmnopqrstuvwxyz'
        result = ''
        for i in range(length):
            result += chars[random.randrange(len(chars))]
        return result

    def generate_csrf_token():
        if '_csrf_token' not in session:
            session['_csrf_token'] = random_str()
        return session['_csrf_token']

    app.jinja_env.globals['csrf_token'] = generate_csrf_token

在HTML模板中直接使用 `csrf_token()`就可以生成CSRF TOKEN.


### 获取model的表名, 对象的dict

有的时候需要在程序里面获取model的表名，可以这么做：

    from sqlalchemy import inspect

    inspect(User).mapped_table.name


另外，我们可能需要把model对象serialize成JSON对象，可以使用inspect来获取对象的
dict：

    import json
    from sqlalchemy import inspect

    class User(Base):
        id = Column(Integer, primary_key=True, nullable=False, autoincrement=True)
        type = Column(Enum('mobile', 'flow', 'card', 'other'), nullable=False, default='other')
        price = Column(Integer, nullable=False)
        title = Column(String(255), nullable=False, default='')

        def to_json(self):
            inspector = inspect(self)
            od = copy.deepcopy(inspector.dict) # 获取对象的dict
            del od['_sa_instance_state'] # 删除不必要的字段
            od['start_time'] = str(od['start_time']) # datetime对象转成string
            od['end_time'] = str(od['end_time'])
            json_str = json.dumps(od)
            return json_str
