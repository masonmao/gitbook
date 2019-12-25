# 初识Nginx

![1576424538312](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576424538312.png)

可扩展性好 - 模块化设计

高可靠性 -- 运行数年不需重启

热部署 -- 在不停止服务的情况下升级nginx

BSD许可证 -- 开源可修改源码

## nginx的组成

![1576424669410](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576424669410.png)

需要驾驶员

二进制可执行文件 -- 官方模块和第三方模块编译而成

nginx.conf -- 驾驶员，定义处理请求的行为

access.log -- 汽车的gps轨迹，处理信息和请求信息

error.log -- 黑匣子，定位问题

## 版本发布历史

![1576424816157](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576424816157.png)

feature -- 新增功能

bugfix -- 修复bug

change -- 做的重构

main live 主干版本

stable 稳定版本

## 版本选择

![1576424993530](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576424993530.png)

![1576425019764](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576425019764.png)

![1576425048584](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576425048584.png)

api服务器和web防火墙 openresty是个不错的选择

其他情况使用开源的Nginx就足够了

## 编译过程

直接安装 -- 默认配置文件，可能不会加载一些第三方模块

编译安装-- 可指定第三方模块

### 安装包目录

![1576425183937](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576425183937.png)

![1576425275603](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576425275603.png)

![1576425286307](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576425286307.png)

cc 用于编译

changes 版本特性和bugfix

changes.ru 俄罗斯版本,作者是俄罗斯人

conf 示例文件

configure

contrib 两个脚本和vim工具，vim工具可以在vim展示nginx语法高亮

html 包括两个htmp文件，50x.html报500错误，index.html欢迎页面

man linux对nginx的帮助文件

src 源代码

### 编译

![1576425649098](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576425649098.png)

configure的参数

with指定加载的模块

without 默认加载的模块，指定后移除

configure --prefix=path 指定安装目录

configure执行完成后生成中间文件，在objs目录下

ngx_module.c 记录了所有加载的模块

make之后在objs目录生成nginx二进制文件

最后指定make install

## nginx的配置语法

![](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576425962955.png)

![1576462206343](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576462206343.png)

![1576462221531](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576462221531.png)

![1576462233057](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576462233057.png)

http模块

upstream 表示上有服务

server域名

location 对应url

## nginx命令行

![1576462316312](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576462316312.png)

![1576462426031](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576462426031.png)

重载配置文件 nginx -s reload 不停止服务的前提下，重新加载配置文件

热部署 -- 

1）新版Nginx编译后的二进制文件，拷贝到nginx的安装目录，替换旧的nginx二进制文件

2）给Master进程发信号 kill -USR2 pid，此时会新起一个master进程，旧的master此时还在运行；新的master会生成新的worker,与旧的worker并存

3）将所有请求平滑过渡到新的Nginx服务中,新的连接和请求会到新的master中

4）kill -WINCH pid 优雅的关掉旧的master进程,旧worker已经关掉，旧的master不会被杀死，支持reload进行版本回退

切割日志文件

nginx -s reopen重新生成日志文件

## 配置静态资源服务器

![1576463332393](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576463332393.png)

autoindex 给用户展示目录结构

set $limit_rate 1k; 限制访问速度，每秒传输x字节给浏览器

### 日志格式

![1576463485957](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576463485957.png)

log路径存在server下

## 搭建具备缓存功能的反向代理服务

![1576465717218](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576465717218.png)

缓存目录

![1576465766312](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576465766312.png)

proxy_cache_valid 

## goAccess图形化实时可视化

![1576466042392](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576466042392.png)

![1576466152521](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576466152521.png)

## ssl安全协议

![1576466224156](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576466224156.png)

![1576466284390](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576466284390.png)

密钥交换

身份验证

数据加密/强度和模式 GCM提高多核性能

## 对称和非对称加密

![1576466372125](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576466372125.png)

发送方用密钥将明文加密，接收方用密钥将加密文档解密

![1576466431068](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576466431068.png)

+是异或操作，加密和解密用的是相同的密钥

![1576466527296](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576466527296.png)

一对密钥：公钥和私钥，性能较差

公钥加密后只有私钥可以解密，私钥加密后只有公钥才可以解密

## ssl证书公信力

![1576466677387](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576466677387.png)

![1576466793720](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576466793720.png)

DV:验证域名的归属

OV:验证机构组织名称是否正确

EV:

## tls协议通讯

![1576467037409](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576467037409.png)

1）浏览器向服务器发送client hello消息，告诉服务器支持的加密算法

2）server hello 服务器告诉浏览器选择哪个安全套件

3）服务器将公钥证书发给浏览器

4）服务器发送server hello done到浏览器

5）生成私钥后将公钥发送给服务器

6）服务器根据自己的私钥和浏览器的公钥生成加密的密钥，浏览器根据自己的私钥和服务器的公钥生成密钥；两个密钥是一样的，由非对称加密算法保证

7）根据密钥进行数据发送和通信

![1576467056386](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576467056386.png)

![1576467067958](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576467067958.png)



![1576467080257](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576467080257.png)

小文件考验非对称加密算法性能--椭圆加密算法

大文件考研对称加密算法--AES算法

![1576467096142](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576467096142.png)



![1576467108480](C:\Users\maoxiangxin\AppData\Roaming\Typora\typora-user-images\1576467108480.png)