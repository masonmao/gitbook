# nginx架构

## nginx请求处理流程

![1576477340804](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576477340804.png)

三种流量，三种状态机，epoll异步处理引擎

## 进程结构

![1576477482176](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576477482176.png)

master-worker多进程模型

master -- worker进程的管理者

worker进程从头到尾占用一个cpu，worker与cpu核绑定

进程间通信通过共享内存实现

进程结构实例

## ![1576477679289](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576477679289.png)

reload和hub

![1576477756324](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576477756324.png)

nginx许多子命令都是通过想Master进程发送信号实现的

## 使用信号管理父子进程

![1576477871626](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576477871626.png)

usr2和winch通过kill命令向master进程发送；其余的可直接通过nginx命令发送

## reload配置文件的真相

![1576478147196](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576478147196.png)

子进程会继承父进程打开的所有端口

![1576478302109](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576478302109.png)

 新的连接在新的worker子进程

异常时，请求长时间占用旧的worker进程，nginx新版本中提供配置项，worker_shut_down_time_out

## 热升级完整流程

![1576478517729](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576478517729.png)

1）中只替换二进制文件

![1576478648523](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576478648523.png)

新的Master是老master的子进程

## 优雅的关闭worker进程

![1576478730355](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576478730355.png)

  

## 网络收发与nginx事件的对应关系

![1576479494778](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576479494778.png)

传输层：记录本地浏览器打开的端口和远端服务器的端口

网络层：记录本机ip和服务器ip

链路层：经过以太网到家里路由器

![1576479612131](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576479612131.png)

数据链路层：在数据前后加上Mac地址

![1576479675700](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576479675700.png)

事件收集/分发器

## 网络时间实例演示

![1576479859236](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576479859236.png)

三次握手引发读事件

## nginx时间驱动模型

![1576479967779](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576479967779.png)

1）nginx刚启动时，epoll_wait等待事件

2）操作系统收到建立连接的报文，且处理成功后

3）操作系统通知epoll_wait，唤醒nginx进程处理

3）操作系统将准备好的事件放入事件队列，nginx从队列中取事件

4）处理事件

5）处理完成后将响应接口写入队列，让内核处理

6）所有事件处理完成后，重新回到epoll_wait

如果处理事件太长，会引发队列内消息积压

## epoll优劣及原理

![1576480316738](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576480316738.png)

横轴--句柄数

![1576480356764](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576480356764.png)

链表中都是活跃的连接

poll和select会遍历所有的请求，耗时较长

操作事件服务都log(n)

## nginx请求切换

![1576480524792](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576480524792.png)

apache 是采用阻塞机制, 且一个进程只处理一个请求,如图如果process1 处理 header请求网络事件不满足,则切换到process2 去执行绿色请求,如果绿色请求依然不满足,就会切换到process3 去执行橙色请求. 每个进程之间切换需要消耗5ms时间,当请求并发增加时,消耗时间成指数性增加,进程之间切换的消耗就不可忽视.容易导致阻塞.

nginx 在时间片时间内(在5ms-800ms之间),当请求网络事件不满足时,nginx直接在当前process内切换请求,直到分配的时间片消耗完毕. 所以一般我们会在nginx配置时,把worker的优先级调到最高-19,(优先级数字在-20~20之间,-20为最高).从而使worker 分配的时间片尽可能多, 从而尽可能少的切换process,让cpu少做无用功.


## 同步&异步 阻塞&非阻塞

![1576481569165](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576481569165.png)

accept队列为空时，会阻塞，可以设置阻塞超时时间

![1576481621348](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576481621348.png)

由代码决定accept收到eagain错误码后的处理逻辑，导致代码逻辑非常复杂

![1576481683973](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576481683973.png)

## nginx的模块

![1576493618778](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576493618778.png)

需要了解

1）提前编译进二进制文件

objs目录下 nginx_module.c ，包含所有被编译到Nginx中的模块

![1576493788161](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576493788161.png)

2）模块提供哪些配置项

![1576493897992](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576493897992.png)

模块源文件中 ngx_command_t结构体的每个成员都是支持的指令名成

![1576493944299](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576493944299.png)

![1576493976482](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576493976482.png)

高内聚，抽象好

3）模块在什么情况被使用

4）该模块提供了哪些变量

## nginx模块分类

![1576494124936](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576494124936.png)

core_module 可以重新定义出新的子模块

![1576494300547](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576494300547.png)

## nginx连接池

![1576494395970](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576494395970.png)

http://nginx.org/en/docs/ngx_core_module.html#worker_connections

每个worker进程都有一个Ngx_cycle_t数据结构

关注三个变量：

1）connections 连接数， 默认512，每个连接自动对应一个读事件和写事件

2）connection_n个读事件

3）connection_n个写事件

![1576494663453](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576494663453.png)

每个连接在64位操作系统占用的字节数为232字节

一个事件的字节数96字节

所以一个连接的字节数大约为 232+96*2

timer是基于红黑树实现的定时器

sent变量 已经发送的字节数，可在nginx官方文档中查看

## 内存池对性能的影响

![1576495014651](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576495014651.png)

nginx内存碎片比较小

提前分配内存，小块内存从内存池取出，大块内存通过allocate分配

**连接内存池**: 连接建立时分配内存，断开时释放

![1576495339025](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576495339025.png)

连接时保存的上下文信息比较少

**请求内存池**：通常分配4k保存Header信息

![1576495315063](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576495315063.png)

请求时需要保存大量的上下文信息，如url等

## worker进行协同工作的关键

![1576495438865](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576495438865.png)

信号：管里nginx进程

共享内存：数据同步，开辟一块内存，多个worker进程可同时访问和写入

多个进程同时访问所以需要加锁：自旋锁。1号进程没释放时，2号进程会不停的请求。要求所有进程快速取锁和释放锁，不然会导致思索和性能问题。

![1576495635322](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576495635322.png)

![1576495730610](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576495730610.png)

## slab内存管理器

![1576495813998](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576495813998.png)

## Nginx容器

![1576496186132](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576496186132.png)

![1576496228427](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576496228427.png)name是key,value是个链表

![1576496291033](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576496291033.png)

适用静态不变的内容，nginx启动时就可以确定哈希表的元素

max_size：最大的哈希表bucket个数

bucket_size：桶数量

### 红黑树

![1576496520262](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576496520262.png)

查找二叉树

平衡的

![1576496584178](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576496584178.png)

