# mysql 查询数据

标签（空格分隔）： mysql

---

[toc]

## 建表规范

### Normal Format, NF
- 每个表保存⼀个实体信息
- 每个具有⼀个ID字段作为主键
- ID主键 + 原⼦表

### 1NF, 第⼀范式
> 字段不能再分，就满⾜第⼀范式。

### 2NF, 第⼆范式
> 满⾜第⼀范式的前提下，不能出现部分依赖。
> 消除符合主键就可以避免部分依赖。增加单列关键字。

### 3NF, 第三范式
> 满⾜第⼆范式的前提下，不能出现传递依赖。
> 某个字段依赖于主键，⽽有其他字段依赖于该字段。这就是传递依赖。
> 将⼀个实体信息的数据放在⼀个表内实现。

## 查询语句

```
SELECT [ALL|DISTINCT] select_expr FROM -> WHERE -> GROUP BY [合计函数] -> HAVING -> ORDER BY -> LIMIT
```

### `select_expr`

- 可以⽤ `*` 表⽰所有字段。
    
    ```
    select * from tb;
    ```
- 可以使⽤表达式（计算公式、函数调⽤、字段也是个表达式）
    
    ```
    select stu, 29+25, now() from tb;
    ```
- 可以为每个列使⽤别名。适⽤于简化列标识，避免多个列标识符重复。
    
    ```    
    -- 使用 as 关键字，也可省略 as.
    select stu+10 as add10 from tb;
    ```
### `FROM` ⼦句
> ⽤于标识查询来源。

- 可以为表起别名。使⽤as关键字。

    ```
    SELECT * FROM tb1 AS tt, tb2 AS bb;
    ```
- from⼦句后，可以同时出现多个表。多个表会横向叠加到⼀起，⽽数据会形成⼀个笛卡尔积。

    ```
    SELECT * FROM tb1, tb2;
    ```
- 向优化符提⽰如何选择索引

    ```
    USE INDEX、IGNORE INDEX、FORCE INDEX
    SELECT * FROM table1 USE INDEX (key1,key2) WHERE key1=1 AND key2=2 AND key3=3;
    SELECT * FROM table1 IGNORE INDEX (key3) WHERE key1=1 AND key2=2 AND key3=3;
    ```
### `WHERE` ⼦句

- 从from获得的数据源中进⾏筛选。
- 整型1表⽰真，0表⽰假。
- 表达式由运算符和运算数组成。
- 运算数：变量（字段）、值、函数返回值
- 运算符：`=`, `<=>`, `<>`, `!=`, `<=`, `<`, `>=`, `>`, `!`, `&&`, `||`

### `GROUP BY` ⼦句, 分组⼦句

- `GROUP BY 字段/别名 [排序⽅式]`
- 分组后会进⾏排序。升序：`ASC`，降序：`DESC`
- count 返回不同的⾮NULL值数⽬ `count(*)`、`count(字段)`
- 以下[**合计函数**]需配合 GROUP BY 使⽤：
    - `sum` 求和
    - `max` 求最⼤值
    - `min` 求最⼩值
    - `avg` 求平均值
- `group_concat` 返回带有来⾃⼀个组的连接的⾮NULL值的字符串结果。组内字符串连接。

### `HAVING` ⼦句，条件⼦句
> 与 where 功能、⽤法相同，执⾏时机不同。

|HAVING|WHERE|
|---|---|
|对筛选出的结果再次进行过滤|在开始时执行监测数据，对原数据进行过滤|
|字段必须是查询出来的|字段必须是数据表存在的|
|可以使用字段的别名|不可以使用字段的别名|
|可以使用合计函数|不可以使用合计函数|

### `ORDER BY` ⼦句，排序⼦句

```
order by 排序字段/别名 排序⽅式 [,排序字段/别名 排序⽅式]...
```

- 升序：`ASC`
- 降序：`DESC`

### `LIMIT` ⼦句，限制结果数量⼦句
> 仅对处理好的结果进⾏数量限制。将处理好的结果的看作是⼀个集合，按照记录出现的顺序，索引从0开始。

```
-- 省略第一个参数，表示从索引0开始。limit 获取条数
limit 起始位置, 获取条数
```

### `DISTINCT`, ALL 选项
> 去除重复记录。默认为`all`, 全部记录。

### `UNION`
> 将多个select查询的结果组合成⼀个结果集合。

```
-- 建议，对每个SELECT查询加上小括号包裹。
SELECT ... 
-- 默认 DISTINCT 方式，即所有返回的行都是唯一的
UNION [ALL|DISTINCT] 
SELECT ...
```

## ⼦查询
> ⼦查询需⽤括号包裹。

### `from` 型
> from后要求是⼀个表，必须给⼦查询结果取个别名。

- 简化每个查询内的条件。
- from型需将结果⽣成⼀个临时表格，可⽤以原表的锁定的释放。
- ⼦查询返回⼀个表，表型⼦查询。

```
select * from (select * from tb where id > 0) as subfrom where id > 1;
```

### `where` 型

- ⼦查询返回⼀个值，标量⼦查询。
- 不需要给⼦查询取别名。
- where⼦查询内的表，不能直接⽤以更新。

```
select * from tb where money = (select max(money) from tb);
```

### 列⼦查询
> 如果⼦查询结果返回的是⼀列。
> 使⽤ `in` 或 `not in` 完成查询。
> `exists` 和 `not exists` 条件，如果⼦查询返回数据，则返回1或0。常⽤于判断条件。

```
select column1 from t1 where exists (select * from t2);
```

### ⾏⼦查询
> 查询条件是⼀个⾏。

```
select * from t1 where (id, gender) in (select id, gender from t2);
```

```
-- 行构造符：通常用于与对能返回两个或两个以上列的子查询进行比较。
(col1, col2, ...) 
或 
ROW(col1, col2, ...)
```

### 特殊运算符

- `!= all()` 相当于 `not in`
- `= some()` 相当于 `in`。`any` 是 `some` 的别名
- `!= some()` 不等同于 `not in`，不等于其中某⼀个。
- `all`, `some` 可以配合其他运算符⼀起使⽤。

## 连接查询(join)
> 将多个表的字段进⾏连接，可以指定连接条件。

### 内连接(inner join)
> on 表⽰连接条件。其条件表达式与where类似。也可以省略条件（表⽰条件永远为真）。
> 也可⽤where表⽰连接条件。还有 using, 但需字段名相同。 using(字段名)。

- 默认就是内连接，可省略`inner`。
- 只有数据存在时才能发送连接。即连接结果不能出现空⾏。

### 交叉连接 cross join
> 即，没有条件的内连接。

```
select * from tb1 cross join tb2;
```

### 外连接(outer join)
> 如果数据不存在，也会出现在连接结果中。

#### 左外连接 `left join`
> 如果数据不存在，左表记录会出现，⽽右表为null填充

#### 右外连接 `right join`
> 如果数据不存在，右表记录会出现，⽽左表为null填充

### ⾃然连接(natural join)
> ⾃动判断连接条件完成连接。
> 相当于省略了`using`，会⾃动查找相同字段名。

```
natural join
natural left join
natural right join
```