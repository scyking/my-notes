# Redis 实现接口幂等性

标签（空格分隔）： redis

---

[toc]

## 概念

幂等性，客户端多次发起同一个请求，必须保证操作只能执行一次。

## 解决方案

- 唯一索引 防止新增脏数据
- token机制 防止页面重复提交
- 悲观锁 获取数据的时候加锁（锁表或者锁行）
- 乐观锁 基于版本号，在更新数据时校验数据
- 分布式锁 redis、zookeeper等
- 状态机 状态变更，更新数据时判断状态


## token机制实现
> 通过 redis + token 机制实现接口幂等性校验。

### 思路

> 每⼀次请求创建⼀个唯⼀标识token，将此token存⼊redis。
> 请求接⼝时,将此token作为参数传给后端，后端判断redis中是否存在此token:
    - 如果存在, 正常处理业务逻辑, 并从redis中删除此token
    - 如果不存在, 说明参数不合法或者是重复请求, 返回提⽰即可
    
### 流程图

```flow
st=>start: 开始
op=>operation: 客户端获取token，发起请求
op1=>operation: 服务端获取参数中token
op3=>operation: 校验token
op4=>operation: 处理业务逻辑
cond=>condition: 判断redis中token是否存在
out1=>inputoutput: 删除token
out2=>inputoutput: 重复请求
ed=>end: 结束

st->op->op1->cond
cond(no)->out1->ed
cond(yes)->op3->op4->out2->ed
```



