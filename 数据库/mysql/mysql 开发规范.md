# mysql 开发规范

标签（空格分隔）： mysql

---

[toc]

## 概述
> 整理自《Alibaba Java开发手册》。

## 规范

### 命名相关

1. 所有的数据库对象名称必须使⽤⼩写字⺟并⽤下划线分割（MySQL⼤⼩写敏感，名称要⻅名知意，最好不超过32字符）
1. 所有的数据库对象名称禁⽌使⽤MySQL保留关键字（如 `desc`、`range`、`match`、`delayed` 等，请参考 MySQL官⽅保留字 ）
1. 临时库表必须以`tmp`为前缀并以⽇期为后缀（`tmp_`）
1. 备份库和库必须以`bak`为前缀并以⽇期为后缀(`bak_`)

### 表相关

1. 所有存储相同数据的列名和列类型必须⼀致。（在多个表中的字段如`user_id`，它们类型必须⼀致）
1. mysql5.5之前默认的存储的引擎是myisam，没有特殊要求。更高版本，所有的表必须使⽤innodb（innodb好处⽀持失误，⾏级锁，⾼并发下性能更好，对多核，⼤内存，ssd等硬件⽀持更好）
1. 数据库和表的字符集尽量统⼀使⽤`utf8`（字符集必须统⼀，避免由于字符集转换产⽣的乱码，汉字utf8下占3个字节）
1. 所有表和字段都要添加注释`COMMENT`，从⼀开始就进⾏数据字典的维护
1. 尽量控制单表数据量的⼤⼩在500w以内，超过500w可以使⽤历史数据归档，分库分表来实现（500万⾏并不是MySQL数据库的限制。过⼤对于修改表结构，备份，恢复都会有很⼤问题。MySQL没有对存储有限制，取决于存储设置和⽂件系统）
1. 谨慎使⽤mysql分区表（分区表在物理上表现为多个⽂件，在逻辑上表现为⼀个表）
1. 谨慎选择分区键，跨分区查询效率可能更低
1. 建议使⽤物理分表的⽅式管理⼤数据
1. 尽量做到冷热数据分离，减⼩表的宽度（mysql限制最多存储4096列，⾏数没有限制，但是每⼀⾏的字节总数不能超过`65535`。列限制好处：减少磁盘io，保证热数据的内存缓存命中率，避免读⼊⽆⽤的冷数据）
1. 禁⽌在表中建⽴预留字段（⽆法确认存储的数据类型，对预留字段类型进⾏修改，会对表进⾏锁定）
1. 限制每张表上的索引数量，建议单表索引不超过5个（索引会增加查询效率，但是会降低插⼊和更新的速度）
1. 避免建⽴冗余索引和重复索引（冗余：`index（a,b,c)` `index(a,b)` `index(a)`）
1. 禁⽌在数据中存储图⽚，⽂件⼆进制数据（使⽤⽂件服务器）
1. 禁⽌给表中的每⼀列都建⽴单独的索引
1. 每个innodb表必须有⼀个主键，选择⾃增id（不能使⽤更新频繁的列作为主键，不适⽤`UUID`,`MD5`,`HASH`,字符串列作为主键）
1. 区分度最⾼的列放在联合索引的最左侧
1. 尽量把字段⻓度⼩的列放在联合索引的最左侧
1. 在`varchar`字段上建⽴索引时，必须指定索引⻓度，没必要对全字段建⽴索引，根据实际⽂本区分度决定索引⻓度即可。
1. 尽量避免使⽤外键（禁⽌使⽤物理外键，建议使⽤逻辑外键）
1. 优先选择符合存储需要的最⼩数据类型
1. 优先使⽤⽆符号的整形来存储
1. 优先选择存储最⼩的数据类型（`varchar(N)`,N代表的是字符数，⽽不是字节数，N代表能存储多少个汉字）
1. 避免使⽤`Text`或是`Blob`类型
1. 避免使⽤`ENUM`数据类型（修改`ENUM`值需要使⽤`ALTER`语句，`ENUM`类型的`ORDER BY`操作效率低，需要额外操作，禁⽌使⽤数值作为`ENUM`的枚举值
1. 尽量把所有的字段定义为`NOT NULL`（索引`NULL`需要额外的空间来保存，所以需要暂⽤更多的内存，进⾏⽐较和计算要对
`NULL`值做特别的处理）
1. 使⽤`timestamp`或`datetime`类型来存储时间
1. 同财务相关的⾦额数据，采⽤`decimal`类型（不丢失精度，禁⽌使⽤ `float` 和 `double`）
1. 尽量不要使⽤物理删除（即直接删除，如果要删除的话提前做好备份），⽽是使⽤逻辑删除，使⽤字段`delete_flag`做逻辑删除，类型为`tinyint`，0表⽰未删除，1表⽰已删除）

### SQL相关

1. 避免使⽤双`%`号和`like`，搜索严禁左模糊或者全模糊（如果需要请⽤搜索引擎来解决。索引⽂件具有 `B-Tree` 的最左前缀匹配特性，如果左边的值未确定，那么⽆法使⽤此索）
1. 建议使⽤预编译语句进⾏数据库操作
1. 禁⽌跨库查询（为数据迁移和分库分表留出余地，降低耦合度，降低⻛险）
1. 禁⽌`select *` 查询（消耗更多的cpu和io及⽹络带宽资源，⽆法使⽤覆盖索引）
1. 禁⽌使⽤不含字段列表的`insert`语句（不允许`insert into t values（‘a’，‘b’，‘c’）`）
1. `in` 操作能避免则避免，若实在避免不了，需要仔细评估 `in` 后边的集合元素数量，控制在 1000 个之内
1. 禁⽌使⽤`order by rand（）`进⾏随机排序
1. 禁⽌`where`从句中对列进⾏函数转换和计算（例如：`where date（createtime）=‘20160901’` 会⽆法使⽤`createtime`列上索引。改成 `where createtime>='20160901' and createtime <'20160902'`）
1. 尽量使⽤ `union all` 代替 `union`
1. 拆分复杂的⼤SQL为多个⼩SQL（ MySQL⼀个SQL只能使⽤⼀个CPU进⾏计算）
1. 尽量避免使⽤⼦查询，可以把⼦查询优化为`join`操作（⼦查询的结果集⽆法使⽤索引，⼦查询会产⽣临时表操作，如果⼦查询数据量⼤会影响效率，消耗过多的CPU及IO资源）
1. 超过100万⾏的批量写操作，要分批多次进⾏操作（⼤批量操作可能会造成严重的主从延迟，binlog⽇志为row格式会产⽣⼤量的⽇志，避免产⽣⼤事务操作）
1. 对于⼤表使⽤`pt—online-schema-change`修改表结构（避免⼤表修改产⽣的主从延迟，避免在对表字段进⾏修改时进⾏锁表）
1. 超过三个表禁⽌ `join`。（需要 `join`的字段，数据类型必须绝对⼀致；多表关联查询时，保证被关联的字段需要有索引。即使
双表 `join` 也要注意表索引、SQL 性能。）
1. 使⽤ `ISNULL()`来判断是否为 NULL 值。
1. 如果有 `order by` 的场景，请注意利⽤索引的有序性。`order by` 最后的字段是组合,索引的⼀部分，并且放在索引组合顺序的最后，避免出现 `file_sort` 的情况，影响查询性能。
1. 在代码中写分⻚查询逻辑时，若 `count` 为 0 应直接返回，避免执⾏后⾯的分⻚语句
1. SQL 性能优化的⽬标：⾄少要达到 `range` 级别，要求是 `ref` 级别，如果可以是 `consts` 最好

### 其他

1. 对于程序连接数据库账号，遵循权限最⼩原则
1. 禁⽌在线上做数据库压⼒测试
1. 禁⽌从开发环境，测试环境直接连⽣产环境数据库




