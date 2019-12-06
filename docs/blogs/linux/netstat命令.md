# netstat命令详解

## 输出信息

```shell

[root@sy-suz-srv51 ~]# netstat
Active Internet connections (w/o servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 k8sdev.sui:sun-sr-https k8sdev.suiyi.com.:34880 SYN_RECV
tcp        0      0 k8sdev.suiyi.com.c:2379 10.1.62.21:47910        ESTABLISHED
tcp        0      0 k8sdev.suiyi.com.c:2379 k8sdev.suiyi.com.:37790 ESTABLISHED
tcp        0      0 sy-suz-srv:pcsync-https 10.1.62.162:49200       ESTABLISHED
tcp        0      0 k8sdev.suiyi.com.:52866 k8sdev.sui:sun-sr-https ESTABLISHED
tcp        0      0 k8sdev.suiyi.com.:37728 k8sdev.suiyi.com.c:2379 ESTABLISHED
tcp        0      0 k8sdev.sui:sun-sr-https k8sdev.suiyi.com.:52852 ESTABLISHED
tcp        0      0 k8sdev.sui:sun-sr-https 10.1.62.162:32841       ESTABLISHED
tcp        0      0 sy-suz-srv:pcsync-https sy-suz-srv51:60094      ESTABLISHED
tcp        0      0 localhost:webcache      localhost:40136         ESTABLISHED
tcp        0      0 k8sdev.suiyi.com.:35466 10.1.62.21:sun-sr-https ESTABLISHED
tcp        0      0 k8sdev.suiyi.com.:34358 10.1.62.21:sun-sr-https ESTABLISHED
Active UNIX domain sockets (w/o servers)
Proto RefCnt Flags       Type       State         I-Node   Path
unix  3      [ ]         DGRAM                    18442    /run/systemd/notify
unix  2      [ ]         DGRAM                    18444    /run/systemd/cgroups-agent
unix  2      [ ]         DGRAM                    23822    /var/run/chrony/chronyd.sock
unix  8      [ ]         DGRAM                    18455    /run/systemd/journal/socket
unix  18     [ ]         DGRAM                    18457    /dev/log
unix  2      [ ]         DGRAM                    14151    /var/run/nscd/socket
unix  2      [ ]         DGRAM                    584      /run/systemd/shutdownd
unix  3      [ ]         STREAM     CONNECTED     124439388 /run/dbus/system_bus_socket
unix  3      [ ]         STREAM     CONNECTED     42312    /run/systemd/journal/stdout
unix  3      [ ]         STREAM     CONNECTED     39909
unix  3      [ ]         STREAM     CONNECTED     21675
unix  3      [ ]         STREAM     CONNECTED     47538
unix  3      [ ]         STREAM     CONNECTED     124585242 /var/run/docker/containerd/docker-containerd.sock
unix  3      [ ]         STREAM     CONNECTED     21658
unix  2      [ ]         STREAM     CONNECTED     30160
unix  3      [ ]         STREAM     CONNECTED     33750    /run/systemd/journal/stdout
unix  3      [ ]         STREAM     CONNECTED     124614293 @/containerd-shim/moby/c44e49ee0f86d8a4109afb176701795c64f44655abb1861275bbd3b2a9f76394/shim.sock
unix  3      [ ]         STREAM     CONNECTED     124609611 @/containerd-shim/moby/a736ba153c07f0bbf099ae1a1069530e35bfa28ae93f8f235d6c35a6c5ed9ce7/shim.sock
unix  3      [ ]         STREAM     CONNECTED     124601653 @/containerd-shim/moby/20d3fd59d03455d45b1da2636fca25d0edd79dac1947c17045a797eb8506157c/shim.sock
```

netstat的输出结果可以分为两个部分

1、Active Internet connections 有源TCP连接，其中"Recv-Q"和"Send-Q"指接收队列和发送队列。这些数字一般都应该是0。如果不是则表示软件包正在队列中堆积。这种情况只能在非常少的情况见到。

2、Active UNIX domain sockets 有源Unix域套接口(和网络套接字一样，但是只能用于本机通信，性能可以提高一倍)。

列名解释：

Proto：显示连接使用的协议。

RefCnt：表示连接到本套接口上的进程号。

Types：显示套接口的类型。

State：显示套接口当前的状态。

Path：表示连接到套接口的其它进程使用的路径名。

## 主要参数

-a (all) 显示所有选项，默认不显示LISTEN相关。
-t (tcp) 仅显示tcp相关选项。
-u (udp) 仅显示udp相关选项。
-n 拒绝显示别名，能显示数字的全部转化成数字。
-l 仅列出有在 Listen (监听) 的服务状态。

-p 显示建立相关链接的程序名
-r 显示路由信息，路由表
-e 显示扩展信息，例如uid等
-s 按各个协议进行统计
-c 每隔一个固定时间，执行该netstat命令。
LISTEN和LISTENING的状态只有用-a或者-l才能看到。

## 常见操作

1）查看连接状态

 netstat -ap | grep httpd | awk '{printf $6}'

 2）查看连接数

netstat -an |grep ESTABLISH | grep "192.168.1.10:80"

3）统计nginx日志中访问最多的五个ip

cat access.log | awk '{print $1}' | uniq -c | sort -n | tail -5