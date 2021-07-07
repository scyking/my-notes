# mysql 触发器

标签（空格分隔）： mysql

---

[toc]

## 概述
> 触发程序是与表有关的命名数据库对象，当该表出现特定事件时，将激活该对象。
> 监听：记录的增加、修改、删除。

## 创建触发器
```
CREATE TRIGGER trigger_name 
trigger_time trigger_event
ON tbl_name
-- 表示任何一条记录上的操作满足触发事件都会触发该触发器。
FOR EACH ROW 
trigger_stmt
```

### 参数

- `trigger_time`：触发程序的动作时间。
    - `before`，指明触发程序是在激活它的语句之后触发。
    - `after`，指明触发程序是在激活它的语句之后触发。
- `trigger_event`：指明了激活触发程序的语句的类型
    - `INSERT`，将新⾏插⼊表时激活触发程序
    - `UPDATE`，更改某⼀⾏时激活触发程序
    - `DELETE`，从表中删除某⼀⾏时激活触发程序
- `tbl_name`：监听的表，必须是永久性的表，不能将触发程序与TEMPORARY表或视图关联起来。
- `trigger_stmt`：当触发程序激活时执⾏的语句。执⾏多个语句，可使⽤`BEGIN...END`复合语句结构

### NEW与OLD
> MySQL 中定义了 `NEW` 和 `OLD`，用来表示触发器的所在表中，触发了触发器的那一行数据，来引用触发器中发生变化的记录内容。

- 更新操作，更新前是`OLD`，更新后是`NEW`.
- 删除操作，只有`OLD`.
- 增加操作，只有`NEW`.

### 注意事项
> 对于具有相同触发程序动作时间和事件的给定表，不能有两个触发程序。

## 查看
```
-- 显示所有触发器的基本信息，无法查询指定的触发器。
SHOW TRIGGERS
-- 在information_schema.triggers表中查看触发器信息
SELECT * FROM information_schema.triggers
```

## 删除
>如果不需要某个触发器时一定要将这个触发器删除，以免造成意外操作，这很关键。
```
DROP TRIGGER [IF EXISTS] [schema_name.]trigger_name
```

## 特殊的执⾏
1. 只要添加记录，就会触发程序。
2. `Insert into on duplicate key update` 语法会触发：
    - 如果没有重复记录，会触发 `before insert`, `after insert`;
    - 如果有重复记录并更新，会触发 `before insert`, `before update`, `after update`;
    - 如果有重复记录但是没有发⽣更新，则触发 `before insert`, `before update`;
3. `Replace` 语法 如果有记录，则执⾏ `before insert`, `before delete`, `after delete`, `after insert`




