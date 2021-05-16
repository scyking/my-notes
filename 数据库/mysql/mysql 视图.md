# mysql 视图

标签（空格分隔）： mysql

---

[toc]

## 概念
> 视图是⼀个虚拟表，其内容由查询定义。
> 同真实的表⼀样，视图包含⼀系列带有名称的列和⾏数据。
> 但是，视图并不在数据库中以存储的数据值集形式存在。
> ⾏和列视图具有表结构⽂件，但不存在数据⽂件。

## 创建视图

```
CREATE [OR REPLACE] [ALGORITHM = {UNDEFINED | MERGE | TEMPTABLE}] VIEW view_name [(column_list)] AS select_statement
```

- 视图名必须唯⼀，同时不能与表重名。
- 视图可以使⽤`select`语句查询到的列名，也可以⾃⼰指定相应的列名。
- 可以指定视图执⾏的算法，通过`ALGORITHM`指定。
- `column_list`如果存在，则数⽬必须等于`SELECT`语句检索的列数

### 查看结构
```
SHOW CREATE VIEW view_name
```

### 删除视图

```
DROP VIEW [IF EXISTS] view_name ...
```

- 删除视图后，数据依然存在。
- 可同时删除多个视图。

### 修改视图结构
> ⼀般不修改视图，因为不是所有的更新视图都会映射到表上。

```
ALTER VIEW view_name [(column_list)] AS select_statement
```

## 视图作⽤

1. 简化业务逻辑
2. 对客⼾端隐藏真实的表结构

## 视图算法(ALGORITHM)

### MERGE 合并
> 将视图的查询语句，与外部查询需要先合并再执⾏！

### TEMPTABLE 临时表
> 将视图执⾏完毕后，形成临时表，再做外层查询！

### UNDEFINED 未定义(默认)
> 指的是MySQL⾃主去选择相应的算法。