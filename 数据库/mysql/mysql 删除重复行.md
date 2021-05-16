# mysql 删除重复行

标签（空格分隔）： mysql

---

[toc]

## 查找重复行

- `test`表结构如下

    ```
    +----+------------+
    | id | day |
    +----+------------+
    | 1 | 2016-10-08 |
    | 2 | 2016-10-08 |
    | 3 | 2016-10-09 |
    +----+------------+
    ```

- 根据具有相同值的字段分组，然后知显⽰⼤⼩⼤于1的组。

    ```
    select day, count(*) from test group by day HAVING count(*) > 1;
    ```

PS：不能使用`WHERE`子句，因为`WHERE`⼦句过滤的是分组之前的⾏，`HAVING`⼦句过滤的是分组之后的⾏。

## 删除重复行

### 使用临时表

```
create temporary table to_delete (day date not null, min_id int not null);

insert into to_delete(day, min_id)
    select day, MIN(id) from test group by day having count(*) > 1;
```

- `to_delete`临时表结构如下

    ```
    +------------+--------+
    | day | min_id |
    +------------+--------+
    | 2016-10-08 | 1 |
    +------------+--------+
    ```
    
- 删除重复数据

    ```
    delete from test
        where exists(
            select * from to_delete
            where to_delete.day = test.day and to_delete.min_id <> test.id
        )
    ```