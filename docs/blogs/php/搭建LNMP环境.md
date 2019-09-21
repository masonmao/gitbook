# 搭建LNMP环境

[TOC]

入职这么久，总是用现成的环境，突然想自己搭建一套lnmp环境试试。

## 安装nginx

### 下载并解压

```shell
[root@08a4912c2dd7 /]# cd /usr/local/src/
[root@08a4912c2dd7 src]# wget http://nginx.org/download/nginx-1.12.2.tar.gz
[root@08a4912c2dd7 src]# tar zxvf nginx-1.12.2.tar.gz
```

### 取消debug编译模式

```shell
[root@08a4912c2dd7 src]# cd nginx-1.12.2
[root@08a4912c2dd7 nginx-1.12.2]# vim auto/cc/gcc
```

将下图中172行注释掉

```shell
168 # stop on warning
169 CFLAGS="$CFLAGS -Werror"
170
171 # debug
172 #CFLAGS="$CFLAGS -g"
173
174 # DragonFly's gcc3 generates DWARF
175 #CFLAGS="$CFLAGS -g -gstabs"
```

### 预编译

```shell
[root@08a4912c2dd7 nginx-1.12.2]# ./configure --prefix=/usr/local/nginx --with-http_ssl_module --with-http_v2_module --with-http_stub_status_module --with-pcre --with-http_gzip_static_module --with-http_dav_module   --with-http_addition_module  --with-http_sub_module --with-http_flv_module  --with-http_mp4_module
```

其中：

--with-http_gzip_static_module ：支持压缩

--with-http_stub_status_module ：支持nginx状态查询

--with-http_ssl_module ：支持https

--with-pcre ：为了支持rewrite重写功能，必须制定pcre

 --with-http_dav_module             #启用支持（增加PUT,DELETE,MKCOL：创建集合，COPY和MOVE方法）                        
--with-http_addition_module         #启用支持（作为一个输出过滤器，支持不完全缓冲，分部分相应请求）
--with-http_sub_module              #启用支持（允许一些其他文本替换Nginx相应中的一些文本）
--with-http_flv_module              #启用支持（提供支持flv视频文件支持）
--with-http_mp4_module              #启用支持（提供支持mp4视频文件支持，提供伪流媒体服务端支持）

### 编译

```shell
[root@08a4912c2dd7 nginx-1.12.2]#  make && make install
```

### 添加系统变量

这一步是为了保证服务的启动和停止

```shell
[root@08a4912c2dd7 local]# vim /etc/profile
#在配置文件中添加如下代码
export PATH=/usr/local/nginx/sbin:$PATH
```

重启配置文件

```shell
[root@08a4912c2dd7 local]# source /etc/profile
```

### 生成服务启动脚本

```shell
[root@08a4912c2dd7 nginx-1.12.2]# vim /usr/local/nginx/load.sh
```

```shell
p#!/bin/bash
# chkconfig: - 99 2
# description: Nginx Service Control Script
PROG="/usr/local/nginx/sbin/nginx"
PIDF="/usr/local/nginx/logs/nginx.pid"
case "$1" in
        start)
        $PROG
        ;;
        stop)
   id     kill -3 $(cat $PIDF)
        ;;
        restart)
        $0 stop &> /dev/null
        if [ $? -ne 0 ] ; then continue ; fi
        $0 start
        ;;
        reload)
        kill -1 $(cat $PIDF)
        ;;
        *)
        echo "Userage: $0 { start | stop | restart | reload }"
        exit 1
esac
exit 0
```

其中

/usr/local/nginx/sbin/nginx是nginx程序的执行路径

pid文件是文本文件，只有一行，记录了该进程的pid。作用就是防止进程启动多个副本，只有获得pid文件写入权限的进程才能正常启动并将进程号写入pid文件中。

## 安装Mysql

### 下载并解压

```shell
[root@08a4912c2dd7 src]# wget http://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.17-linux-glibc2.5-x86_64.tar.gz
[root@08a4912c2dd7 src]# tar -zxvf mysql-5.7.17-linux-glibc2.5-x86_64.tar.gz
[root@08a4912c2dd7 src]# mv mysql-5.7.17-linux-glibc2.5-x86_64 mysql
```

直接看这个吧：	https://www.jianshu.com/p/5a2a92059d8e

## 安装php

