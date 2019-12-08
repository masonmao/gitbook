# php-fpm详解

PHP-FPM(PHP FastCGI Process Manager)：PHP FastCGI 进程管理器，提供了更好的php进程管理方式，可以有效控制内存和进程，并可以实现php的平滑重载。在php5.3.3之后，php-fpm已经被官方收录。

## 作用

管理php fastcgi进程。

那么什么是fastcgi呢？在介绍fastcgi之前，有必要先说一下什么是cgi.

## CGI

cgi（common gateway interface）可以理解为一种协议约定。在静态html时代，web服务器只是简单的处理浏览器的http请求，将存储在服务器上的html文件返回给浏览器。随着技术的发展，网站也越来越复杂，简单的请求响应已完全不能满足需求。而我们知道，服务器有不能直接运行php这样的文件，因此，cgi应运而生。cgi只是接口协议，接收约定好的输入参数，返回标准格式的结果。

![image-20190901195947323](/Users/maoxiangxin/Library/Application Support/typora-user-images/image-20190901195947323.png)

### cgi工作原理

当浏览器请求web服务器时，web服务器会请求操作系统创建一个cgi解释器进程。创建cgi进程时，总会去php.ini中读取配置信息，初始化运行环境。而且cgi在处理完一个请求之后会马上退出，等下次请求到来时会重新创建cgi进程。

不难发现，在互联网发展的早期，数据请求量较少，这样每次都创建进程的做法可能不会导致什么问题。但是随着访问量的不断增加，并发量不断增大，这样频繁的执行创建进程，会导致巨大的开销，严重降低效率，于是就产生了fast-cgi。

## FAST-CGI

fastcgi可以理解成是一个常驻进程的cgi。它只需要读取一次配置文件，无需每次请求都创建，大大提高服务器的响应速度。

### FAST-CGI工作原理

　　**1.Web Server启动时载入FastCGI进程管理器（IIS ISAPI或Apache Module)**

​        **2.FastCGI进程管理器自身初始化，启动多个CGI解释器进程(可见多个php-cgi)并等待来自Web Server的连接。**

​        **3.当客户端请求到达Web Server时，FastCGI进程管理器选择并连接到一个CGI解释器。 Web server将CGI环境变量和标准输入发送到FastCGI子进程php-cgi。**

​       **4.FastCGI 子进程完成处理后将标准输出和错误信息从同一连接返回Web Server。当FastCGI子进程关闭连接时， 请求便告处理完成。FastCGI子进程接着等待并处理来自FastCGI进程管理器(运行在Web Server中)的下一个连接。 在CGI模式中，php-cgi在此便退出了。**

## PHP-FPM

php-cgi只是个CGI程序，他自己本身只能解析请求，返回结果，不会进程管理。Php-fpm就是用来介绍上面提到的多个fast-cgi进程。PHP-FPM 会创建一个主进程，控制何时以及如何将HTTP请求转发给一个或多个子进程处理。PHP-FPM主进程还控制着什么时候创建(处理Web应用更多的流量)和销毁(子进程运行时间太久或不再需要了)PHP子进程。PHP-FPM进程池中的每个进程存在的时间都比单个HTTP请求长,可以处理10、50、100、500或更多的HTTP请求。

Php-fpm的配置文件

```php
[global]
pid = run/php-fpm.pid
error_log = log/php-fpm.log
log_level = notice
#错误级别. 可用级别为: alert（必须立即处理）, error（错误情况）, warning（警告情况）, notice（一般重要信息）, debug（调试信息）. 默认: notice.

rlimit_files = 65535
#设置核心rlimit最大限制值.

[www]
user = joy
group = joy
listen = 127.0.0.1:9000
#fpm监听端口，即nginx中php处理的地址

listen.backlog = 2048
#backlog数，-1表示无限制，由操作系统决定，此行注释掉就行。

pm = dynamic
pm.max_children = 1024
#，子进程最大数

pm.start_servers = 10         
#控制服务启动时创建的进程数

pm.min_spare_servers = 10     
#，保证空闲进程数最小值，如果空闲进程小于此值，则创建新的子进程

pm.max_spare_servers = 60
#保证空闲进程数最大值，如果空闲进程大于此值，此进行清理

pm.max_requests = 102400
#设置每个子进程重生之前服务的请求数. 对于可能存在内存泄漏的第三方模块来说是非常有用的. 如果设置为 '0' 则一直接受请求. 等同于 PHP_FCGI_MAX_REQUESTS 环境变量. 默认值: 0.

request_terminate_timeout = 10s
#设置单个请求的超时中止时间. 该选项可能会对php.ini设置中的'max_execution_time'因为某些特殊原因没有中止运行的脚本有用. 设置为 '0' 表示 'Off'.当经常出现502错误时可以尝试更改此选项。

request_slowlog_timeout = 10s
#当一个请求该设置的超时时间后，就会将对应的PHP调用堆栈信息完整写入到慢日志中. 设置为 '0' 表示 'Off'

slowlog = var/log/$pool.log.slow
#慢请求的记录日志,配合request_slowlog_timeout使用
```

