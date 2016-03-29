---
layout: post
title: 'Django测试fixtures使用'
date: 2014-04-25
author: Pengfei.X
version: 0.1
categories: [python, ]
excerpt: 使用fixtures给测试数据库填充数据
---

上午给统计项目做了一次更新，结果Admin界面挂了，之前也有过更新代码导致出现BUG，
当时也没有给予关注，忽略了给项目增加基本的测试，事不过三，今天出现这样的问题后，
甚感惭愧，紧急修复了问题，而且正好赶上抢小米路由，结果还抢到了。


回到正题，给Django添加测试用例很简单，在APP下加个tests.py，里面写一些测试用例，
继承Django的Test类就可以了，

    from django.test import TestCase, Client

    class FeedTestCase(TestCase):
        fixtures = ['product.json', 'reason.json', 'reasonorder.json', 'users.json']

        def setUp(self):
            self.c = Client()

写了两个测试，结果出现数据库相关的错误，原来是测试数据里面是空的，没有具体的数据，
需要实现填充，搜了一下文档，可以通过增加`fixtures`属性，在测试的时候，自动填充数据。
具体做法为，在APP下面建一个文件夹`fixtures`，然后，dump测试需要使用到的数据，
    
    python manage.py dumpdata --indent=2 --format=json auth.User > feed/fixtures/users.json

在测试类中，增加`fixtures`，如上代码所示，ok了。

    开跑，结果出现了错误，导入数据的时候，有些字符数据库不认识，需要更改配置文件，

    OperationalError: (1366, "Incorrect string value: '\\xE2\\x80\\xA8<br...' for column 'html' at row 1")

更新配置，在default数据库的配置中，增加一项：

    'TEST_CHARSET': 'UTF8',

OK, 跑通。


## 参考 ##

1. [Django Test Driven Development](http://www.realpython.com/blog/python/django-1-6-test-driven-development/#.U1ntaXWSyHc)
2. [Error in django unittest while loading a fixture](http://stackoverflow.com/questions/3819693/error-in-django-unittest-while-loading-a-fixture)
3. [test-charset](https://docs.djangoproject.com/en/dev/ref/settings/#test-charset)
