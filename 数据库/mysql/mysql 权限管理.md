# mysql 权限管理

标签（空格分隔）： mysql

---

[toc]

## root密码重置
> ⽤⼾信息表：`mysql.user`。

1. 停⽌MySQL服务
2. 跳过授权表

    ```
    [Linux] /usr/local/mysql/bin/safe_mysqld --skip-grant-tables
    [Windows] mysqld --skip-grant-tables
    ```
3. `use mysql;`
4. `UPDATE user SET PASSWORD=PASSWORD("密码") WHERE user = "root";`
5. `FLUSH PRIVILEGES;`

## 刷新权限

```
FLUSH PRIVILEGES;
```

## 增加⽤⼾
```
CREATE USER 用户名 IDENTIFIED BY [PASSWORD] 密码(字符串)
```

- 必须拥有mysql数据库的全局`CREATE USER`权限，或拥有`INSERT`权限。
- 只能创建⽤⼾，不能赋予权限。
- ⽤⼾名，注意引号：如 `'user_name'@'192.168.1.1'`
- 密码也需引号，纯数字密码也要加引号
- 要在纯⽂本中指定密码，需忽略`PASSWORD`关键词。要把密码指定为由`PASSWORD()`函数返回的混编值，需包含关键字`PASSWORD`

## 重命名⽤⼾
```
RENAME USER old_user TO new_user
```

## 设置密码

```
SET PASSWORD = PASSWORD('密码') -- 为当前⽤⼾设置密码
SET PASSWORD FOR ⽤⼾名 = PASSWORD('密码') -- 为指定⽤⼾设置密码
```

## 删除⽤⼾
```
DROP USER 用户名
```

## 分配权限/添加⽤⼾
```
GRANT 权限列表 ON 表名 TO 用户名 [IDENTIFIED BY [PASSWORD] 'password']
```

- `all privileges` 表⽰所有权限
- `*.*` 表⽰所有库的所有表
- `库名.表名` 表⽰某库下⾯的某表

    ```
    GRANT ALL PRIVILEGES ON `pms`.* TO 'pms'@'%' IDENTIFIED BY 'pms0817';
    ```
    
## 查看权限
```
SHOW GRANTS FOR 用户名
```

## 查看当前⽤⼾权限

```
SHOW GRANTS; 
SHOW GRANTS FOR CURRENT_USER; 
SHOW GRANTS FOR CURRENT_USER();
```

## 撤消权限
```
REVOKE 权限列表 ON 表名 FROM 用户名
REVOKE ALL PRIVILEGES, GRANT OPTION FROM 用户名 -- 撤销所有权限
```

## 权限层级
> 要使⽤GRANT或REVOKE，您必须拥有`GRANT OPTION`权限，并且您必须⽤于您正在授予或撤销的权限。

### 全局层级
> 全局权限适⽤于⼀个给定服务器中的所有数据库，`mysql.user`。

```
-- 授予全局权限
GRANT ALL ON *.*
-- 撤销全局权限
REVOKE ALL ON *.*
```

### 数据库层级
> 数据库权限适⽤于⼀个给定数据库中的所有⽬标，`mysql.db`, `mysql.host`。

```
-- 授予数据库权限
GRANT ALL ON db_name.*
-- 撤销数据库权限
REVOKE ALL ON db_name.*
```

### 表层级
> 表权限适⽤于⼀个给定表中的所有列，`mysql.talbes_priv`。

```
-- 授予表权限
GRANT ALL ON db_name.tbl_name
-- 撤销表权限
REVOKE ALL ON db_name.tbl_name
```

### 列层级
> 列权限适⽤于⼀个给定表中的单⼀列，`mysql.columns_priv`。
> 当使⽤`REVOKE`时，您必须指定与被授权列相同的列。

## 权限列表
```
ALL [PRIVILEGES]            -- 设置除GRANT OPTION之外的所有简单权限
ALTER                       -- 允许使用ALTER TABLE
ALTER ROUTINE               -- 更改或取消已存储的子程序
CREATE                      -- 允许使用CREATE TABLE
CREATE ROUTINE              -- 创建已存储的子程序
CREATE TEMPORARY TABLES     -- 允许使用CREATE TEMPORARY TABLE
CREATE USER                 -- 允许使用CREATE USER, DROP USER, RENAME USER和REVOKE ALL PRIVILEGES。
CREATE VIEW                 -- 允许使用CREATE VIEW
DELETE                      -- 允许使用DELETE
DROP                        -- 允许使用DROP TABLE
EXECUTE                     -- 允许用户运行已存储的子程序
FILE                        -- 允许使用SELECT...INTO OUTFILE和LOAD DATA INFILE
INDEX                       -- 允许使用CREATE INDEX和DROP INDEX
INSERT                      -- 允许使用INSERT
LOCK TABLES                 -- 允许对您拥有SELECT权限的表使用LOCK TABLES
PROCESS                     -- 允许使用SHOW FULL PROCESSLIST
REFERENCES                  -- 未被实施
RELOAD                      -- 允许使用FLUSH
REPLICATION CLIENT          -- 允许用户询问从属服务器或主服务器的地址
REPLICATION SLAVE           -- 用于复制型从属服务器（从主服务器中读取二进制日志事件）
SELECT                      -- 允许使用SELECT
SHOW DATABASES              -- 显示所有数据库
SHOW VIEW                   -- 允许使用SHOW CREATE VIEW
SHUTDOWN                    -- 允许使用mysqladmin shutdown
SUPER                       -- 允许使用CHANGE MASTER, KILL, PURGE MASTER LOGS和SET GLOBAL语句，mysqladmin debug命令；允许您连接（一次），即使已达到max_connections
UPDATE                      -- 允许使用UPDATE
USAGE                       -- “无权限”的同义词
GRANT OPTION                -- 允许授予权限
```


