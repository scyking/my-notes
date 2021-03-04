# mycat 配置文件详解

标签（空格分隔）： mycat

---

## 核心配置文件

### `schema.xml` 说明

管理这mycat的逻辑库、表、分片规则、`DataNode`以及`DataSource`。

1. `schema` 标签：定义逻辑库（可划分多个逻辑库）。

    |属性名|值|数量限制|描述|
    |:---:|:---:|:---:|:---:|
    

1. `table` 标签：定义逻辑表，所有需要拆分的表都需要在这个标签中定义。


1. `childTable` 标签：定义E-R分片的子表。


1. `dataNode` 标签：定义数据节点（数据分片）。


1. `dataHost` 标签：定义具体的数据库实例、读写分离配置和心跳语句。

    |属性名|值|数量限制|描述|
    |:---:|:---:|:---:|:---:|
    |name|String|1|唯一标识，供上层标签使用|
    |maxCon|Integer|1|指定每个读写实例连接池的最大连接|
    |minCon|Integer|1|指定每个读写实例连接池的最小连接|
    |balance|Integer|1||
    |writeType|Integer|1||
    |dbType|String|1|指定后端连接的数据库类型|
    |dbDriver|String|1|指定连接后端数据库使用的 Driver|
    |switchType|Integer|||
    |tempReadHostAvailable|Integer|||
    
    - **balance** 属性取值
        - `0`，不开启读写分离机制，所有**读操作**都发送到当前可用的 `writeHost`上。
        - `1`，全部的 `readHost` 与 `stand by writeHost` 参与 `select` 语句的负载均衡，简单的说，当双
主双从模式(M1->S1，M2->S2，并且 M1 与 M2 互为主备)，正常情况下，M2,S1,S2 都参与 select 语句的负载
均衡。
        - `2`，所有读操作都随机的在 `writeHost`、`readhost` 上分发。
        - `3`，所有读请求随机的分发到 `wiriterHost` 对应的 `readhost` 执行，`writerHost` 不负担读压力。（PS：仅1.4+版本支持该取值）
        
    - **writeType** 属性取值
        - `0`， 所有写操作发送到配置的第一个 `writeHost`，第一个挂了切到还生存的第二个
`writeHost`，重新启动后以切换后的为准，切换记录在配置文件中 `dnindex.properties` 。
        - `1`，所有写操作都随机的发送到配置的 `writeHost`。（PS： 1.5 以后废弃不推荐）
        
    - **switchType** 属性取值
        - `-1`，表示不自动切换
        - `1`，默认值，自动切换
        - `2`，基于mysql主从同步状态决定是否切换。心跳语句为`show slave status`
        -  `3`,基于 MySQL galary cluster 的切换机制。心跳语句为 `show status like 'wsrep%'`
    
1. `heartbeat` 标签：指明用于和后端数据库进行心跳检查的语句。

### `server.xml` 说明

系统配置信息。

### `rule.xml` 说明

定义表拆分规则。



