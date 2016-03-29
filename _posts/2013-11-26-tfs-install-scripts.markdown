---
layout: post
title: 'tfs install scripts'                                     
date: 2013-11-26
author: Pengfei.X
version: 0.1
categories: [others, ]
excerpt: automatically install tfs
---


    rpm -Uvh http://mirror.webtatic.com/yum/el5/latest.rpm
    yum install mysql.`uname -i` yum-plugin-replace
    yum replace mysql --replace-with mysql55
    yum remove mysql mysql-devel
    yum install mysql55 mysql55-server mysql55-devel


    yum install svn -y
    yum install -y libtool.x86_64 libtool-ltdl-devel.x86_64 libtool-ltdl.x86_64
    yum install -y zlib-devel mysql-devel
    yum install -y readline.x86_64 readline-devel.x86_64
    
    
    cd /
    mkdir taobao-series
    cd /taobao-series/
    mkdir source_code
    
    
    cd source_code/
    svn checkout -r 17 http://code.taobao.org/svn/tb-common-utils/trunk/ tb-common-utils
    export TBLIB_ROOT="/taobao-series/lib"
    if grep -Fxqv "TBLIB_ROOT"  ~/.bash_profile; then
        echo 'TBLIB_ROOT="/taobao-series/lib"' >> ~/.bash_profile
    fi
    cd tb-common-utils/
    sh build.sh
    
    
    cd ..
    wget http://downloads.sourceforge.net/project/libuuid/libuuid-1.0.2.tar.gz
    tar zxf libuuid-1.0.2.tar.gz 
    cd libuuid-1.0.2
    ./configure --prefix=/taobao-series/libuuid
    make && make install
    
    
    cd ..
    wget https://github.com/couchbase/gperftools/archive/2.0.1-linux.tar.gz
    tar zxf 2.0.1-linux
    cd gperftools-2.0.1-linux
    ./configure --prefix=/taobao-series/gperftools --enable-frame-pointers
    make && make install
    if [ ! -f /etc/ld.so.conf.d/gperftools.conf ]; then
        echo "/taobao-series/gperftools/lib" >> /etc/ld.so.conf.d/gperftools.conf
    fi
    ldconfig
    
    
    cd ..
    wget http://www.canonware.com/download/jemalloc/jemalloc-3.4.1.tar.bz2
    bunzip2 jemalloc-3.4.1.tar.bz2
    tar xf jemalloc-3.4.1.tar
    cd jemalloc-3.4.1
    ./configure --prefix=/taobao-series/jemalloc
    make && make install
    if [ ! -f /etc/ld.so.conf.d/jemalloc.conf ]; then
        echo "/taobao-series/jemalloc/lib" >> /etc/ld.so.conf.d/jemalloc.conf
    fi
    ldconfig
    
    
    cd ..
    svn co http://code.taobao.org/svn/tfs/tags/release-2.2.10 
    cd release-2.2.10/
    ./build.sh init
    ./configure --prefix=/taobao-series/tfs --with-uuid=/taobao-series/libuuid --with-tcmalloc=/taobao-series/gperftools --with-jemalloc=/taobao-series/jemalloc
    make && make install
    
    rm -rf /taobao-series/source_code
