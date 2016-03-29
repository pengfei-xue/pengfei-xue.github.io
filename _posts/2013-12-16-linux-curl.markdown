---
layout: post
title: 'linux curl'
date: 2013-12-16
author: Pengfei.X
version: 0.1
categories: [others, ]
excerpt: curl  is  a  tool  to transfer data from or to a server, using one of the supported protocols (DICT, FILE, FTP, FTPS, GOPHER, HTTP, HTTPS, IMAP, IMAPS, LDAP, LDAPS, POP3, POP3S, RTMP, RTSP, SCP, SFTP,  SMTP,  SMTPS,  TELNET and TFTP).  The command is designed to work without user interaction.
---


做web开发很多时候需要测试一下接口工作是否正常，HTTP请求的来回数据是否是我们想要的,
使用curl可以很方便地做到这一点，也不需要写程序来做，简单方便。但是，curl的选项有些
多，经常忘记，需要不时地使用man来常看选项的意思。这篇文章算是自己使用curl的一个小
总结吧，以后可以找到一个参考的地方，用到了新的会继续更新，:)。

## curl 常用的选项示例 ##


- POST 方法

`-X` 指定HTTP METHOD, 默认是GET 

`-H` 添加HTTP Header 

`--data` HTTP POST的Body数据 


request:

    curl -X POST -H "IAMAHEADER: header" --data "username=xxx&passowrd=xxx" http://httpbin.org/post


response:

    {
        "json": null,
        "files": {},
        "form": {
            "passowrd": "xxx",
            "username": "xxx"
        },
        "args": {},
        "data": "",
        "url": "http://httpbin.org/post",
        "origin": "19.36.6.2",
        "headers": {
            "Content-Length": "25",
            "Connection": "close",
            "Iamaheader": "header",
            "Accept": "*/*",
            "User-Agent": "curl/7.29.0",
            "Content-Type": "application/x-www-form-urlencoded",
            "Host": "httpbin.org"
        }
    }



- Cookie

`-c, --cookie-jar <file name>` 把cookie写入文件中
`-b, --cookie <name=data>` 发送请求时，带上cookie信息, 如果不是name=data的形式，则从文件中读取


request:
    
    # 把cookie写入 cookies.txt中
    curl -L -c cookies.txt http://httpbin.org/cookies/set?k1=v1&k2=v2

    # 读取cookie
    curl -b cookie.txt http://httpbin.org/cookies


response:
    
    {
        "cookies": {
            "k1": "v1" 
        }
    }



- 测试下载速度

`-w` Defines  what  to  display  on stdout after a completed and successful operation.


request:

    curl -r 0-1048576 -L -w %{speed_download} -o/dev/null -s "http://g-ec2.images-amazon.com/images/G/28/kindle/Pinot_Final/ImageBlock/kp_slate_lg_01_final_CN._V366214291_.jpg"
    


- 通过代理带问

`-x --proxy <[protocol://][user:password@]proxyhost[:port]>` Use the specified HTTP proxy. 

  
request:

    curl https://facebook.com -x http://127.0.0.1:8080 -v
