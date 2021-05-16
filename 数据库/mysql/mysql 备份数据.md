# mysql 备份数据

标签（空格分隔）： mysql

---

[toc]

## 数据导入导出

```
-- 导出表数据
select * into outfile 文件地址 [控制格式] from 表名; 
-- 导入数据，生成的数据默认的分隔符是制表符
load data [local] infile 文件地址 [replace|ignore] into table 表名 [控制格式]; 
```

- `local` 未指定，则数据⽂件必须在服务器上
- `replace` 和 `ignore` 关键词控制对现有的唯⼀键记录的重复的处理

### 控制格式
> `fields` 控制字段格式。

```
-- 默认
fields terminated by '\t' enclosed by '' escaped by '\\'
```

- 终止 `terminated by 'string'`
- 包裹 `enclosed by 'char'`
- 转义 `escaped by 'char'` 

```
-- 示例：
SELECT a,b,a+b INTO OUTFILE '/tmp/result.text'
FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"'
LINES TERMINATED BY '\n'
FROM test_table;
```

- `lines` 控制⾏格式（默认：`lines terminated by '\n'`）
- 终⽌ `terminated by 'string'` 

## 清空数据 TRUNCATE

```
TRUNCATE [TABLE] tbl_name
```

|TRUNCATE（清空数据）|DELETE（删除）|
|---|---|
|删除表再创建|逐条删除|
|重置`auto_increment`的值|不会|
|不知道删除了几条|知道|
|当被用于带分区的表时，会保留分区|不会|

## 备份与还原
> 备份，将数据的结构与表内数据保存起来。利⽤ `mysqldump` 指令完成。

### 导出

```
mysqldump [options] db_name [tables]
mysqldump [options] ---database DB1 [DB2 DB3...]
mysqldump [options] --all--database
```

1. 导出⼀张表

    ```
    mysqldump -u 用户名 -p 密码 库名 表名 > (D:/a.sql)
    ```
2. 导出多张表

    ```
    mysqldump -u 用户名 -p 密码 库名 表1 表2 表3 > 文件名(D:/a.sql)
    ```
3. 导出所有表

    ```
    mysqldump -u 用户名 -p 密码 库名 > 文件名(D:/a.sql)
    ```
4. 导出⼀个库

    ```
    mysqldump -u 用户名 -p 密码 --lock-all-tables --database 库名 > ⽂件名(D:/a.sql)
    可以-w携带WHERE条件
    ```

### 导⼊
1. 在登录mysql的情况下：
    > source 备份⽂件

2. 在不登录的情况下
    > `mysql -u 用户名 -p 密码 库名 < 备份文件`

