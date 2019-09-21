# mysql分布式事务XA详解

[TOC]

## 1. 事务和分布式事务的概念

### 事务

​		一个数据库事务通常包含了一个序列的对数据库的读/写操作。当事务被提交给了[DBMS](https://baike.baidu.com/item/DBMS)（[数据库管理系统](https://baike.baidu.com/item/数据库管理系统)），则[DBMS](https://baike.baidu.com/item/DBMS)（[数据库管理系统](https://baike.baidu.com/item/数据库管理系统)）需要确保该事务中的所有操作都成功完成且其结果被永久保存在数据库中，如果事务中有的操作没有成功完成，则事务中的所有操作都需要被[回滚](https://baike.baidu.com/item/回滚)，回到事务执行前的状态;同时，该事务对数据库或者其他事务的执行无影响，所有的事务都好像在独立的运行，即事务之间具有隔离性。

### 分布式事务

​		上面提到的事务通常是针对于同一数据库而言，即所有的读写操作都在同一个数据库内完成。但是当一个序列的操作包含多个不同的数据库时，我们如何保证事务的一致性呢？下面就引出了分布式事务的概念。

​		分布式事务指事务的参与者、支持事务的服务器、资源服务器以及事务管理器分别位于不同的分布式系统的不同节点之上。简单的说，一个序列的读写操作包含了很多小的操作，这些小的操作本身可能就是一个事务，这些小的操作分布在不同的数据库服务器上，且属于不同的应用，分布式事务需要保证这些小操作要么全部成功，要么全部失败。本质上来说，分布式事务就是为了保证不同数据库的数据一致性。

## 2. 事务的ACID特性

​		说到事务，我们不得不说一下事务的ACID特性，分布式事务正式为了保证跨库事务的ACID特性才被提出来的。

### 2.1 原子性（A）

​		所谓的原子性就是说，在整个事务中的所有操作，要么全部完成，要么全部不做，没有中间状态。对于事务在执行中发生错误，所有的操作都会被回滚，整个事务就像从没被执行过一样。

### 2.2 一致性（C）

​		事务的执行必须保证系统的一致性，就拿转账为例，A有500元，B有300元，如果在一个事务里A成功转给B50元，那么不管并发多少，不管发生什么，只要事务执行成功了，那么最后A账户一定是450元，B账户一定是350元。

### 2.3 隔离性（I）

​		所谓的隔离性就是说，事务与事务之间不会互相影响，一个事务的中间状态不会被其他事务感知。

### 2.4 持久性（D）

​		所谓的持久性，就是说一单事务完成了，那么事务对数据所做的变更就完全保存在了数据库中，即使发生停电，系统宕机也是如此。

## 3. 分布式事务产生的原因

### 3.1 分库分表

​		像mysql这种关系型书苦苦本身比较容易成为系统瓶颈，单机存储容量、连接数、处理能力都有限。当单表的数据量达到1000W或100G以后，由于查询维度较多，即使添加从库、优化索引，做很多操作时性能仍下降严重。这种我们就需要对数据库进行分库分表，将关联度较低的表分散到多个数据库中（垂直拆分），或者将数据量较大的表按照哈希规则进行分表操作（水平拆分）。

​		数据库分库之后，如果一个操作既需要访问A数据库，又要访问B数据库，我们如何保证数据的一致性呢？这时候就需要使用分布式事务了。

![image-20190720135159250](/Users/maoxiangxin/Library/Application Support/typora-user-images/image-20190720135159250.png)

### 3.2 应用SOA化

​		所谓的SOA化，就是业务的服务化。比如原来单机支撑了整个电商网站，现在对整个网站进行拆解，分离出了订单中心、用户中心、库存中心。对于订单中心，有专门的数据库存储订单信息，用户中心也有专门的数据库存储用户信息，库存中心也会有专门的数据库存储库存信息。这时候如果要同时对订单和库存进行操作，那么就会涉及到订单数据库和库存数据库，为了保证数据一致性，就需要用到分布式事务。

![image-20190720135948214](/Users/maoxiangxin/Library/Application Support/typora-user-images/image-20190720135948214.png)		

## 4. 如何解决分布式事务

Mysql为我们提供了分布式事务解决方案，首先声明两个概念。

- **资源管理器（resource manager）**：用来管理系统资源，是通向事务资源的途径。数据库就是一种资源管理器。资源管理还应该具有管理事务提交或回滚的能力。

- **事务管理器（transaction manager）：**事务管理器是分布式事务的核心管理者。事务管理器与每个资源管理器（resource manager）进行通信，协调并完成事务的处理。事务的各个分支由唯一命名进行标识。

  ​	mysql在执行分布式事务（外部XA）的时候，mysql服务器相当于xa事务资源管理器，与mysql链接的客户端相当于事务管理器。

### 4.1 分布式事务原理：分段式提交

​		分布式事务通常采用2PC协议，全称Two Phase Commitment Protocol。该协议主要为了解决在分布式数据库场景下，所有节点间数据一致性的问题。分布式事务通过2PC协议将提交分成两个阶段：

- prepare；
- commit/rollback

**阶段一为准备（prepare）阶段**。即所有的参与者准备执行事务并锁住需要的资源。参与者ready时，向transaction manager报告已准备就绪。 

**阶段二为提交阶段（commit）**。当transaction manager确认所有参与者都ready后，向所有参与者发送commit命令。

![è¿éåå¾çæè¿°](http://img.blog.csdn.net/20160123165802871)

### 4.2 事务协调者transaction manager

​		因为XA 事务是基于两阶段提交协议的，所以需要有一个事务协调者（transaction manager）来保证所有的事务参与者都完成了准备工作(第一阶段)。如果事务协调者（transaction manager）收到所有参与者都准备好的消息，就会通知所有的事务都可以提交了（第二阶段）。MySQL 在这个XA事务中扮演的是参与者的角色，而不是事务协调者（transaction manager）。

### 4.3 外部XA和内部XA

​		在使用innodb作为存储引擎，并且开启binlog的情况下，MySQL同时维护了binlog日志与innodb的redo log

为了保证这两个日志的一致性，MySQL使用了XA事务，由于只在单机上工作，所以被称为内部XA。

​		外部XA用于跨多MySQL实例的分布式事务，需要应用层作为协调者，通俗的说就是比如我们在PHP中写代码，那么PHP书写的逻辑就是协调者。应用层负责决定提交还是回滚，崩溃时的悬挂事务。MySQL数据库外部XA可以用在分布式数据库代理层，实现对MySQL数据库的分布式事务支持，例如开源的代理工具：网易的DDB，淘宝的TDDL等等

### 4.4  XA事务基本语法

​		XA {START|BEGIN} xid [JOIN|RESUME]     启动一个XA事务 (xid 必须是一个唯一值; [JOIN|RESUME]  字句不被支持)    

​		XA END xid [SUSPEND [FOR MIGRATE]]   结束一个XA事务 ( [SUSPEND [FOR MIGRATE]] 字句不被支持)

​		XA PREPARE xid    准备

​		XA COMMIT xid [ONE PHASE]    提交XA事务

​		XA ROLLBACK xid  回滚XA事务

​		XA RECOVER   查看处于PREPARE 阶段的所有XA事务

### 4.5 测试脚本

```php
<?php

//connect to mysql
$mysql1 = new mysqli("127.0.0.1","root","123456","maoxx_test", 4000) or die("database1 连接失败");
$mysql2 = new mysqli("10.96.78.158","root","123456","zeus_new_shard", 4000)or die("database2 连接失败");

//生成xid，必须唯一
$xid = uniqid("");

//两个库指定同一个事务id，表明这两个库的操作处于同一事务中
$mysql1->query("XA START '$xid'");//准备事务1
$mysql2->query("XA START '$xid'");//准备事务2

try {
    //数据库1执行更新操作
    $return = $mysql1->query("UPDATE tb_member SET name='bbb' WHERE id=1") ;
    if($return == false) {
       throw new Exception("数据库1执行update tb_member操作失败！");
    }
    
    //数据库2执行更新操作
    $return = $mysql2->query("UPDATE es_member_50 SET realname = 'ddd' WHERE id=1") ;
    if($return == false) {
       throw new Exception("数据库2执行update es_member_50操作失败！");
    }

    //phase 1: prepare
    $mysql1->query("XA END '$xid'");
    $mysql1->query("XA PREPARE '$xid'");
   
    $mysql2->query("XA END '$xid'");
    $mysql2->query("XA PREPARE '$xid'");

    //phase 2: commit
    $mysql1->query("XA COMMIT '$xid'");
    $mysql2->query("XA COMMIT '$xid'");
}
catch (Exception $e) {
    //phase 2: rollback
    $mysql1->query("XA ROLLBACK '$xid'");
    $mysql2->query("XA ROLLBACK '$xid'");
    die($e->getMessage());
}

$mysql1->close();
$mysql2->close();
```

### 4.6 XA两阶段提交的不足

1. 性能问题

   XA协议遵循强一致性。在事务执行过程中，各个节点占用着数据库资源，只有当所有节点准备完毕，事务协调者才会通知提交，参与者提交后释放资源。这样的过程有着非常明显的性能问题。

2. 协调者单点故障问题

   事务协调者是整个XA模型的核心，一旦事务协调者节点挂掉，参与者收不到提交或是回滚通知，参与者会一直处于中间状态无法完成事务。

3. 丢失消息导致的不一致问题

   在XA协议的第二个阶段，如果发生局部网络问题，一部分事务参与者收到了提交消息，另一部分事务参与者没收到提交消息，那么就导致了节点之间数据的不一致。