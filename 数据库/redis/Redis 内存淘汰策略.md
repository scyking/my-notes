# Redis 内存淘汰策略

标签（空格分隔）： redis

---

[toc]

## 问题

> 在使⽤内存作为缓存的时候，缓存的⼤⼩⼀般是固定的。当缓存被占满，这个时候继续往缓存⾥⾯添加数据，就需要淘汰⼀部分⽼的数据，释放内存空间⽤来存储新的数据。

## 解决办法

### 增加 redis 可用内存
> 这种方法局限于机器本身内存。

- 通过配置文件配置

    ```
    maxmemory 1000mb
    ```
    
- 通过命令修改

    ```
    config set maxmemory 1000mb
    ```
    
### 过期删除策略
> Redis 使用了**惰性删除与定期删除**，处理过期数据。

- 定时删除：对于设有过期时间的key，时间到了，定时器任务立即执行删除。

> 因为要维护一个定时器，所以就会占用cpu资源，尤其是有过期时间的redis键越来越多损耗的性能就会线性上升。

- 惰性删除：每次只有再访问key的时候，才会检查key的过期时间，若是已经过期了就执行删除。

> 这种情况只有在访问的时候才会删除，所以有可能有些过期的redis键一直不会被访问，就会一直占用redis内存。

- 定期删除：每隔一段时间，就会检查删除掉过期的key。

> 这种方案相当于上述两种方案的折中，通过最合理控制删除的时间间隔来删除key，减少对cpu的资源的占用消耗，使删除操作合理化。

```
# 表示1s执行10次定期删除，即每隔100ms执行一次
hz 10
# 随机抽取过期key数量
maxmemory-samples 5
# 当已用内存超过maxmemory限定时，就会触发主动清理策略
maxmemory 1000mb
```

### 内存淘汰策略
> 补充处理执行**过期删除策略**后，仍未淘汰的key。

1. `noeviction`(默认策略)：对于写请求不再提供服务，直接返回错误（DEL请求和部分特殊请求除外）
1. `allkeys-lru`：从所有key中使⽤LRU算法进⾏淘汰
1. `volatile-lru`：从设置了过期时间的key中使⽤LRU算法进⾏淘汰
1. `allkeys-random`：从所有key中随机淘汰数据
1. `volatile-random`：从设置了过期时间的key中随机淘汰
1. `volatile-ttl`：在设置了过期时间的key中，根据key的过期时间进⾏淘汰，越早过期的越优先被淘汰
1. `volatile-lfu`（Redis4.0 后支持）：从设置了过期时间的key中使⽤LFU算法进⾏淘汰
1. `allkeys-lfu`（Redis4.0 后支持）：从所有key中使⽤LFU算法进⾏淘汰

## 涉及算法

### LRU (Least Recently Used)
> 即**最近最少使⽤**，是⼀种缓存置换算法。

- 核心思想
> 如果⼀个数据在最近⼀段时间没有被⽤到，那么将来被使⽤到的可能性也很⼩，所以就可以被淘汰掉。

- 实现方式
> 采用`哈希表+双向链表`的结构。哈希表用来查询链表中数据的位置，链表负责数据的插入。

    1. 当链表满的时候，将链表尾部的数据丢弃。
    1. 每当缓存命中，将数据移到链表头部。

### LFU (Least Frequently Used)
> 即**最不经常使用**策略。在一段时间内，数据被使用频次最少的，优先被淘汰。

- 核心思想
> 根据key的最近被访问的频率进⾏淘汰，很少被访问的优先被淘汰，被访问的多的则被留下来。

- 实现方式
> 采用`哈希表+两个双向链表`的结构。上方链表用来计数，下方链表用来记录存储数据，哈希表记录下方链表的数据。

    1. 将需要存储的数据插入
    1. 在hash表中**存在**，找到对应的下方双向链表，将该节点的上一个节点和该节点的下一个节点相连，然后判断自己所在上方双向链表的计数是否比当前计数大1
        - **如果是**，则将自己移到该上方双向链表，并且**判断该双向链表下是否还有元素**，如果没有，则要删除该节点
        - **如果不是或者该上方双向列表无下个节点**，则新加节点，将计数设为当前计数+1
    1. 在hash表**不存在**，将数据存入hash表，将数据与双向链表的头节点相连

- 相比LRU算法优点
> LFU算法能更好的表⽰⼀个key被访问的热度。
> 使⽤LRU算法，⼀个key很久没有被访问到，只刚刚是偶尔被访问了⼀次，那么它就被认为是热点数据，不会被淘汰，⽽有些key将来是很有可能被访问到的则被淘汰了。如果使⽤LFU算法则不会出现这种情况，因为使⽤⼀次并不会使⼀个key成为热点数据。

## TTL 实现

### TTL 存储数据结构
> 针对TTL时间，使用`dict *expires`字段存储。
> `dict`是一个`hashtable`，`key`为对应的`rediskey`，`value`为对应的TTL时间。
> `dict`的数据结构中含有2个`dictht`对象，主要是为了解决`hash`冲突过程中重新`hash`数据使用。

### TTL 设置过期时间
> 过期时间最后都会换算到绝对时间进行存储。
> 对于相对时间而言基准时间就是当前时间，对于绝对时间而言相对时间就是0。

- `expire`      按照相对时间且以秒为单位的过期策略
- `expireat`    按照绝对时间且以秒为单位的过期策略
- `pexpire`     按照相对时间且以毫秒为单位的过期策略
- `pexpireat`   按照绝对时间且以毫秒为单位的过期策略