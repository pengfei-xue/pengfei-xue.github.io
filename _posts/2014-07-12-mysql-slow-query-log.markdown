---
layout: post
title: 'mysql slow log query'
date: 2014-04-22
author: Pengfei.X
version: 0.1
categories: [others, ]
excerpt: mysql slow log query
---


### 如何设置

我们经常需要对MYSQL查询进行一些profile，常用到的手段包括EXPALIN查询语句，看看是
否对表进行了全表扫描，没有添加INDEX等;打开MYSQL的慢查询日志功能，在实际生产环境
中检查哪些查询占用了过多的时间等。

打开慢查询日志，可以通过在MYSQL的配置中增加以下几行：

    log_slow_queries    = /var/log/mysql/mysql-slow.log
    long_query_time = 2
    # log-queries-not-using-indexes

第一行是设置慢查询日志的文件地址，第二行为判断慢查询的条件，这里设置的是如果查询
时间超过2秒，就认为是慢查询。但时间环境中，可能一个查询超过200毫秒，就已经很慢了，
需要结合实际的情况来设定。(5.5还不支持把慢查询时间设置成毫秒级别，但是，可以设置
成0, 这样的话，会记录所有的查询)


### 日志解读

生成日志后，需要查看具体的日志，日志格式如下：

    # Time: 140712  9:34:04
    # User@Host: oxx[oxx] @  [192.168.2.161]
    # Query_time: 0.014806  Lock_time: 0.000058 Rows_sent: 0  Rows_examined: 2
    use oxx;
    SET timestamp=1405128844;
    INSERT INTO url (url, hash)
            SELECT 
                  'http://fifa2014.oupeng.com/download?xi=ppeb&v=7' AS url
                , '54876857b9cb32aa8cbadcb37f7ce679' AS hash
            FROM url 
            WHERE NOT EXISTS(
                SELECT *
                FROM url
                WHERE hash = '54876857b9cb32aa8cbadcb37f7ce679'
            )
            LIMIT 1;


第一行是查询发生的时间, 第二行为用户信息，第三行是查询时间，最后是具体的查询语句。


### 分析日志

分析慢查询日志，有很多工具，推荐使用mysqlsla，可以生成很详细的报表，如下：

    Auto-detected logs as slow logs
    Report for slow logs: slowquery.lost20k.log
    1.57k queries total, 19 unique
    Sorted by 't_sum'
    Grand Totals: Time 48 s, Lock 3 s, Rows sent 16.77k, Rows Examined 8.14M


    ______________________________________________________________________ 001 ___
    Count         : 901  (57.54%)
    Time          : 23.810739 s total, 26.427 ms avg, 11.565 ms to 137.291 ms max  (49.46%)
      95% of Time : 20.515587 s total, 23.995 ms avg, 11.565 ms to 58.087 ms max
    Lock Time (s) : 53.756 ms total, 60 �s avg, 43 �s to 230 �s max  (2.13%)
      95% of Lock : 48.123 ms total, 56 �s avg, 43 �s to 105 �s max
    Rows sent     : 0 avg, 0 to 0 max  (0.00%)
    Rows examined : 2 avg, 2 to 2 max  (0.02%)
    Database      : oxx
    Users         : 
        oxx@ 192.168.2.161 : 100.00% (901) of query, 57.54% (901) of all users

    Query abstract:
    SET timestamp=N; INSERT INTO url (url, hash) SELECT 'S' AS url , 'S' AS hash FROM url WHERE NOT EXISTS( SELECT * FROM url WHERE hash = 'S' ) LIMIT N;

    Query sample:
    SET timestamp=1405125515;
    INSERT INTO url (url, hash)
            SELECT 
                  'http://fifa2014.oupeng.com/download?xi=pog9&v=7' AS url
                , '36b64a52e577e8a92d25cecc76a61be9' AS hash
            FROM url 
            WHERE NOT EXISTS(
                SELECT *
                FROM url
                WHERE hash = '36b64a52e577e8a92d25cecc76a61be9'
            )
            LIMIT 1;
