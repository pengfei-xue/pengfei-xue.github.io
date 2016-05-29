---
layout: post
title: 'mysql kill lock queries'
date: 2014-04-22
author: Pengfei.X
version: 0.1
categories: [others, ]
excerpt: mysql show processlist
---

刚才同事说MySQL挂了，从她的应用里面访问不了，跟我说是数据库坏了，让我帮忙恢复一下，
问她是什么状况，也没说清楚，好吧，那就给你恢复一下，从线上dump了一份数据，再scp回来，
折腾了半个小时，网络那个慢，没办法说。

最终拷贝到开发环境的服务器上，开始导入，但是，每次导入都会卡死，有可能是当前数据库已经
处于死锁了，查看了一下文档，发现可以使用`show processlist`来查看当前的查询线程，连接等
信息，如下：

    mysql> show processlist;
    +--------+------+-----------+------+---------+------+----------------+------------------------------------------------------------------------------------------------------+
    | Id     | User | Host      | db   | Command | Time | State          | Info                                                                                                 |
    +--------+------+-----------+------+---------+------+----------------+------------------------------------------------------------------------------------------------------+
    | 639721 | root | localhost | bbs  | Query   |    0 | creating table | CREATE TABLE `wp_links` (
      `link_id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
      `link_url` varc | 
    | 639821 | root | localhost | NULL | Query   |    0 | NULL           | show processlist                                   

mysql提供了`kill`命令来杀掉进行中的查询，命令是`kill [thread id]`, 

    dbh = MySQLdb.Connect(host='192.168.1.xx', user='xxx', passwd='xxx', db='xxx')
    c = dbh.cursor()
    c.execute('show processlist')

    for i in c.fetchall():
        if i[3] == 'xxx':
            dbh.query('kill %s' % i[0])
