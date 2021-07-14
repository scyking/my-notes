# mysql sql优化

标签（空格分隔）：mysql

---

[toc]

## `explain` 命令
> 可对`select`语句分析，并输出`select`执行的详细信息。

```
explain <待分析sql>
```

### 输出列含义

- `id`：查询标识符
- `select_type`：查询类型
- `table`：查询的是哪个表
- `partitions`：匹配的分区
- `type`：join类型
- `possible_keys`：此次查询中可能使用的索引
- `key`：此次查询中使用到的索引
- `key_len`：表示查询用户器使用了索引的字节数。
- `ref`：使用索引涉及字段或常数
- `rows`：此次查询大概扫描行数
- `filtered`：此次查询条件过滤的数据百分比
- `extra`：额外信息

### `select_type`

- `SIMPLE`， 表示此查询不包含 UNION 查询或子查询
- `PRIMARY`， 表示此查询是最外层的查询
- `UNION`， 表示此查询是 UNION 的第二或随后的查询
- `DEPENDENT UNION`， UNION 中的第二个或后面的查询语句， 取决于外面的查询
- `UNION RESULT`， UNION 的结果
- `SUBQUERY`， 子查询中的第一个 SELECT
- `DEPENDENT SUBQUERY`: 子查询中的第一个 SELECT， 取决于外面的查询. 即子查询依赖于外层查询的结果.

### `type`（重要）
> 可判断此次查询时**全表扫描**还是**索引扫描**。性能依次如下：

1. `system`：表中只有一条数据。这个类型是特殊的 `const` 类型。
1. `const`: 针对主键或唯一索引的等值查询扫描，最多只返回一行数据。 `const` 查询速度非常快，因为它仅仅读取一次即可。
1. `eq_ref`: 此类型通常出现在多表的`join`查询，表示对于前表的每一个结果，都只能匹配到后表的一行结果。并且查询的比较操作通常是`=`，查询效率较高。
1. `ref`: 此类型通常出现在多表的`join`查询，针对于非唯一或非主键索引，或者是使用了**最左前缀**规则索引的查询.
1. `range`: 表示使用索引范围查询，通过索引字段范围获取表中部分数据记录。 
1. `index`: 表示全索引扫描(full index scan)
1. `ALL`: 表示全表扫描，这个类型的查询是性能最差的查询之一。

### `key_len`
> 可以判断组合索引是否完全被使用。计算规则如下：

1. 字符串
    - `char(n)`: n 字节长度
    - `varchar(n)`: 如果是 utf8 编码， 则是 `3n + 2`字节; 如果是 utf8mb4 编码， 则是 `4n + 2` 字节
1. 数值类型
    - `TINYINT`: 1字节
    - `SMALLINT`: 2字节
    - `MEDIUMINT`: 3字节
    - `INT`: 4字节
    - `BIGINT`: 8字节
1. 时间类型
    - `DATE`: 3字节
    - `TIMESTAMP`: 4字节
    - `DATETIME`: 8字节
1. 字段属性: `NULL` 属性 占用一个字节. 如果一个字段是 `NOT NULL` 的， 则没有此属性.

### `rows`（重要）
> 可以直观显示SQL执行效率，原则上数值越小越好。

### `extra`

- `using filesort`：表示不能通过索引达到排序效果，需要额外排序操作。建议优化
- `using index`："覆盖索引扫描"， 表示查询在索引树中就可查找所需数据， 不用扫描表数据文件， 往往说明性能不错
- `using temporary`：查询有使用临时表， 一般出现于排序， 分组和多表 join 的情况， 查询效率不高。建议优化

## 查询优化

### 分批处理
> 分批处理不带分⻚参数的查询或者影响⼤量数据的`update`和`delete`操作。

### 操作符`<>`优化

```
select id from orders where amount != 100;
-- 优化后
(select id from orders where amount > 100) 
union all
(select id from orders where amount < 100 and amount > 0)
```

### `OR`优化
> Innodb 引擎`OR`无法使用组合索引。

```
-- 已有组合索引index(mobile_no,user_id)，索引失效
select id，product_name from orders where mobile_no = '13421800407' or user_id = 100;
-- 使用union优化
(select id，product_name from orders where mobile_no = '13421800407') 
union
(select id，product_name from orders where user_id = 100)
```

### `IN`优化

1. IN适合主表⼤⼦表⼩，EXIST适合主表⼩⼦表⼤。由于查询优化器的不断升级，很多场景这两者性能差不多⼀样了。
2. 尝试改为join查询。

```
select id from orders where user_id in (select id from user where level = 'VIP');
-- 使用JOIN
select o.id from orders o 
left join user u 
on o.user_id = u.id 
where u.level = 'VIP';
```

### 不做列运算
> 通常在查询条件列运算会导致索引失效。

```
-- date_format函数会导致这个查询⽆法使⽤索引
select id from order where date_format(create_time，'%Y-%m-%d') = '2019-07-01';
-- 改写后
select id from order where create_time between '2019-07-01 00:00:00' and '2019-07-01 23:59:59';
```

### 避免`Select all`
> 如果不查询表中所有的列，避免使⽤ SELECT *，它会进⾏全表扫描，不能有效利⽤索引。

### `Like`优化
> 前后模糊查询可以使用`fulltext`，但最优是使用 `Elasticsearch`。

```
-- 索引失效
SELECT column FROM table WHERE field like '%keyword%';
-- 优化后
SELECT column FROM table WHERE field like 'keyword%';
```

### `Join`优化
> join的实现是采⽤Nested Loop Join算法，就是通过驱动表的结果集作为基础数据，通过该结数据作为过滤条件到下⼀个表中循环
查询数据，然后合并结果。
> 如果有多个join，则将前⾯的结果集作为循环数据，再次到后⼀个表中查询数据。

1. 驱动表和被驱动表尽可能增加查询条件，满⾜ON的条件⽽少⽤Where，⽤⼩结果集驱动⼤结果集。
2. 被驱动表的join字段上加上索引，⽆法建⽴索引的时候，设置⾜够的Join Buffer Size。
3. 禁⽌join连接三个以上的表，尝试增加冗余字段。

### `Limit`优化
> limit⽤于分⻚查询时越往后翻性能越差。
> 解决的原则：缩⼩扫描范围 

```
如下所⽰：
select * from orders order by id desc limit 1000000，10
-- 先筛选出ID缩⼩查询范围，写法如下：
select * from orders where id > (select id from orders order by id desc limit 1000000， 1) order by id desc limit 0，10
-- 如果查询条件仅有主键ID，写法如下：
select id from orders where id between 1000000 and 1000010 order by id desc
```
