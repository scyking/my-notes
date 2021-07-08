# Redis 介绍

标签（空格分隔）： redis

---

[toc]

## 与 Memcache 对比

|缓存中间件|Memcache|Redis|
|---|---|---|
|数据类型|支持简单数据类型|数据类型丰富|
|数据持久化|不支持|支持|
|主从|不支持|支持|
|分片|不支持|支持|

## QPS高原因

- 大部分操作基于内存，将数据存储在内存中，存取不会受到硬盘IO的限制
- 使用单进程单线程模型的（K，V）数据库，避免频繁上下文切换和锁的竞争
- 数据结构简单，对数据操作也简单
- 使用多路I/O复用模型，为非阻塞IO（多路复用函数epoll/kqueue/evport/select）

## 单线程支持高并发原因

⼏种 I/O 模型
为什么 Redis 中要使⽤ I/O 多路复⽤这种技术呢？
⾸先，Redis 是跑在单线程中的，所有的操作都是按照顺序线性执⾏的，但是由于读写操作等待⽤⼾输⼊或输出都是阻塞的，所
以 I/O 操作在⼀般情况下往往不能直接返回，这会导致某⼀⽂件的 I/O 阻塞导致整个进程⽆法对其它客⼾提供服务，⽽ I/O 多路
复⽤就是为了解决这个问题⽽出现的。

在 I/O 多路复⽤模型中，最重要的函数调⽤就是 select，该⽅法的能够同时监控多个⽂件描述符的可读可写情况，当其中的某些
⽂件描述符可读或者可写时，select ⽅法就会返回可读以及可写的⽂件描述符个数。

Blocking I/O
先来看⼀下传统的阻塞 I/O 模型到底是如何⼯作的：当使⽤ read 或者 write 对某⼀个⽂件描述符（File Descriptor 以下简称 FD)
进⾏读写时，如果当前 FD 不可读或不可写，整个 Redis 服务就不会对其它的操作作出响应，导致整个服务不可⽤。
这也就是传统意义上的，也就是我们在编程中使⽤最多的阻塞模型：

Reactor 设计模式
Redis 服务采⽤ Reactor 的⽅式来实现⽂件事件处理器（每⼀个⽹络连接其实都对应⼀个⽂件描述符）

⽂件事件处理器使⽤ I/O 多路复⽤模块同时监听多个 FD，当 accept、read、write 和 close ⽂件事件产⽣时，⽂件事件处理器就
会回调 FD 绑定的事件处理器。
虽然整个⽂件事件处理器是在单线程上运⾏的，但是通过 I/O 多路复⽤模块的引⼊，实现了同时对多个 FD 读写的监控，提⾼了⽹
络通信模型的性能，同时也可以保证整个 Redis 服务实现的简单。

## 数据结构
Redis的每⼀种数据结构都由⼀个key和value组成，区别在于value存储的值不同。

### 基础数据结构

- String 最基本的数据类型，二进制安全
- Hash String元素组成的字典，适用于存储对象
- List 按照String元素插入顺序排序。具有栈的特性，FIFO
- Set String元素组成的无序集合，通过哈希表实现，不允许重复
- Sorted Set 通过分数来为集合中的成员从小到大的排序

### 复杂数据结构
- Bitmaps 位图，在string的基础上进行位操作，可以实现节省空间的数据结构
- HyperLogLog 用来做基数统计的算法
    1. `Keys[pattern]`：查找所有符合给定模式pattern的key
    2. `SCAN cursor [MATCH pattern] [COUNT count]`
- Geo geospatial，地理空间索引半径查询
- BloomFilter 布隆过滤器

## 编码方式

- raw
- int
- ht
- zipmap
- linkedlist
- ziplist
- intset

## 通用命令

- keys 列出所有key
- exists 判断key是否存在
- del 删除key
- expire，pexpire 设置key过期时间（单位秒）
- expireat，pexpireat 设置key过期时间（单位毫秒）
- ttl，pttl 获取key的过期时间
- persist 移除key过期时间
- type 判断key是什么类型的数据结构（返回基础数据结构）
    


