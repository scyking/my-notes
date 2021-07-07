# mysql 表分区

标签（空格分隔）： mysql

---

[toc]

## 概念

> 表分区就是将一个大表按照mysql提供的几种方式，分成几个小表。

> 日常开发中我们经常会遇到大表的情况，所谓的大表是指存储了百万级乃至千万级条记录的表。这样的表过于庞大，导致数据库在查询和插入的时候耗时太长、性能低下，如果涉及联合查询的情况，性能会更加糟糕。对表进行分区，目的就是减少数据库的负担，提高数据库的效率，通常来讲就是提高表的增删改查效率。
> 分区是将数据分段划分在多个位置存放，可以是同一块磁盘也可以在不同的机器。分区后，表面上还是一张表，但数据散列到多个位置了。应用程序读写的时候操作的还是大表名字，数据库系统自动去组织分区的数据。

## 表分区的功能

- 与单个磁盘或文件系统分区相比，可以存储更多的数据。
- 很容易就能删除不用或者过时的数据。
- 一些查询可以得到极大的优化。
- 涉及到 SUM()/COUNT() 等聚合函数时，可以并行进行。
- IO 吞吐量更大。
- 分区允许可以设置为任意大小的规则，跨文件系统分配单个表的多个部分。实际上，表的不同部分在不同的位置被存储为单独的表。

## 范围分区（Range Partition)
> 通常是使用频率最高的分区，如按月份划分，这样的数据保持均匀性比较好，如果划分的均匀性不是很好，需要考虑其他分区方法。

```
CREATE TABLE table_name(
	....//创建表的格式不变
)
partition by RANGE(table_column)(
	PARTITION p0 VALUES LESS THAN (values1),
	PARTITION p1 VALUES LESS THAN (values2),
	PARTITION p2 VALUES LESS THAN (values3)
	...
)
```

## 列表分区（List Partition）
> 和范围分区差不多，只不过一个是范围，一个是具体数值。

> 当需要明确控制如何将数据进行分区时，采用这种方式。只能进行单列分区，可以讲数据进行分组，比如按城市分区，几个城市放一起。

```
CREATE TABLE table_name(
	....//创建表不变
)
partition by LIST(table_column)(
	PARTITION p0 VALUES IN (0,4,8,12),
	PARTITION p1 VALUES IN (1,5,9,13), 
	PARTITION p2 VALUES IN (2,6,10,14), 
	PARTITION p3 VALUES IN (3,7,11,15)
	...
)
```

## 哈希分区（Hash Partition）
> 如果数据不是那么容易进行划分，通过这种方式就很灵活了。可以将数据均匀的插入到不同的块，在并发时有利于提高效率，当无法用Range分区时，就可以用Hash分区。

```
CREATE TABLE table_name(
	....//创建表不变
)
 PARTITION BY HASH(table_column) 
 PARTITIONS nums;
```



