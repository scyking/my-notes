# mysql 使用

标签：mysql

---

[toc]

## Windows 上安装 mysql 8.x

### 安装步骤
> 使用`zip`方式安装。

1. [下载 mysql](https://dev.mysql.com/downloads/mysql/)
1. 配置环境变量
    - 添加新系统变量（`MYSQL_HOME` 变量值为`zip`解压目录）
    - 在path里添加 `%MYSQL_HOME%\bin;`
1. 初始化([详细介绍](https://dev.mysql.com/doc/refman/5.7/en/data-directory-initialization-mysqld.html))
    进入解压后bin目录下，在cmd下执行【5.6版本及以下包含data目录，不需要执行】
    
    ```
    mysqld --initialize --console
    ```
1. 记录随机产生的密码（用于首次登陆）
1. 安装服务 

    ```
    mysqld --install 服务名 --服务名可省略
    ```
1. 启动服务

    ```
    net start mysql 
    ```

### 删除mysql

1. 开始 --> 控制面板 --> 管理工具 --> 服务
1. 停掉mysql服务
1. 卸载mysql

    ```
    sc delete mysql
    ```
1. 删除注册表
    - 执行regedit，打开注册表
    - 删除如下包含mysql相关的注册表(PS:删除`MySQL/MySQLD`)

        ```
        HKEY_LOCAL_MACHINE/SYSTEM/ControlSet001/Services/Eventlog/Application/MySQL
        HKEY_LOCAL_MACHINE/SYSTEM/ControlSet002/Services/Eventlog/Application/MySQL
        HKEY_LOCAL_MACHINE/SYSTEM/CurrentControlSet/Services/Eventlog/Application/MySQL
        ```

### 问题解决

- 不是有效的win32程序
> 删除bin目录下名为mysqld的文件（大小为0k）。

- Navicat for MySQL 1251错误（不支持身份认证方式）
> 修改加密规则
    ```
    # 修改加密规则（`root`：用户名； `passwor`d：密码； `localhost`：开放的 ip，开放所有 ip 使用 `%`） 
    ALTER USER 'root'@'localhost' IDENTIFIED BY 'password' PASSWORD EXPIRE NEVER; 
    # 更新一下用户的密码 
    ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password'; 
    # 刷新权限 
    FLUSH PRIVILEGES; 
    ```

- Navicat for MySQL 1045错误
    ```
    # 登录
    mysql -u root -p 
    # 显示已有数据库
    show databases；
    # 使用mysql库
    use mysql；
    # 修改密码
    ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'new password';
    # 退出
    quit
    ```

## Linux 上安装 mysql 

### 安装步骤（`rpm`）
        
1. 执行如下命令安装

    ```
    wget http://repo.mysql.com/mysql80-community-release-el7-1.noarch.rpm
    yum localinstall mysql80-community-release-el7-1.noarch.rpm
    yum install mysql-community-server
    ```
    
    安装完成后，默认路径如下：
    
    - 安装路径应： `/usr/share/mysql` 
    - `mysqldump` 文件位置： `/usr/bin/mysqldump`
    - 配置文件路径： `/etc/my.cnf`
    - 数据目录： `/var/lib/mysql` 
    
1. 启动 mysql

    ```
    # 启动
    service mysqld start
    # 查看运行状态
    service mysqld status
    ```
    
1. 修改初始密码

    ```
    # 查看随机生成密码
    grep 'temporary password' /var/log/mysqld.log
    # 登录
    mysql -uroot -p
    # 修改密码（需包含大写字母、小写字母、数字和特殊字符，并且长度不小于8位）
    ALTER USER 'root'@'localhost' IDENTIFIED BY '密码!';
    ```

### 问题解决

<font color='red'>注意版本问题，可以避免大多数问题。</font>

- 报 `Please read "Security" section of the manual to find out how to run mysqld as root!` 错误

    1. 最简单的方式是在执行命令后添加`--user=root`参数。 （强制使用系统`root`用户初始化，不推荐）
    1. 创建系统新用户，执行命令。
    
- 报 `Can't open the mysql.plugin table. Please run mysql_upgrade to create it.` 错误

    进入mysql安装目录，执行 `mysql_install_db` 命令。
    
## 创建用户及权限分配

### 创建数据库

```
create database testdb DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
```

### 创建用户

1. 允许本地 IP 访问 `localhost`, `127.0.0.1`

    ```
    create user 'test'@'localhost' identified by '123456';
    ```

1. 允许外网 IP 访问

    ```
    create user 'test'@'%' identified by '123456';
    ```

1. 修改用户密码

    ```
    update table mysql.user set password=password('new_password') where User='test' and Host='localhost';
    ```

1. 刷新授权

    ```
    flush privileges;
    ```

### 用户授权

1. 授予用户通过外网IP对于该数据库的全部权限

    ```
    grant all privileges on `testdb`.* to 'test'@'%' identified by '123456';
    ```

1. 授予用户在本地服务器对该数据库的全部权限

    ```
    grant all privileges on `testdb`.* to 'test'@'localhost' identified by '123456';
    ```

1. 刷新授权

    ```
    flush privileges;
    ```
    
## 常用指令汇总

### windows服务

```

net start mysql                                                 -- 启动MySQL
sc create mysql binPath= mysqld_bin_path                        -- 创建Windows服务(PS：等号与值之间有空格)
```

### 连接与断开服务器

```
mysql -h [地址] -P [端口] -u [用户名] -p [密码]
SHOW PROCESSLIST                                                -- 显示哪些线程正在运行
SHOW VARIABLES                                                  -- 显示系统变量信息
```

### 数据库操作

```
SELECT DATABASE();                                              -- 查看当前数据库
SELECT now(), user(), version();                                -- 显示当前时间、用户名、数据库版本
CREATE DATABASE[ IF NOT EXISTS] [数据库名] <数据库选项>         -- 创建数据库
        <数据库选项>：
            CHARACTER SET charset_name
            COLLATE collation_name
SHOW DATABASES[ LIKE 'PATTERN']                                 -- 查看已有库
SHOW CREATE DATABASE [数据库名]                                 -- 查看当前库信息
ALTER DATABASE [库名] [选项信息]                                -- 修改库的选项信息
DROP DATABASE[ IF EXISTS] [数据库名]                            -- 删除库
```

### 表操作

```
CREATE [TEMPORARY] TABLE[ IF NOT EXISTS] [库名.]表名 ( 表的结构定义 )[表选项]           -- 创建表
        TEMPORARY                           -- 临时表，会话结束时表⾃动消失
        表的结构定义：
            字段名 数据类型 [NOT NULL | NULL] [DEFAULT default_value] [AUTO_INCREMENT] [UNIQUE [KEY] | [PRIMARY] KEY] 
        表选项:
            CHARSET = charset_name          -- 字符集(如果表没有设定，则使用数据库字符集)
            ENGINE = engine_name            -- 存储引擎
                SHOW ENGINES                            -- 显示存储引擎的状态信息
                SHOW ENGINE [引擎名] {LOGS|STATUS}      -- 显示存储引擎的日志或状态信息
            AUTO_INCREMENT = 行数           -- 自增起始数
            DATA DIRECTORY = '目录'         -- 数据文件目录
            INDEX DIRECTORY = '目录'        -- 索引文件目录
            COMMENT = 'string'              -- 表注释
            PARTITION BY ...                -- 分区选项

SHOW TABLES[ LIKE 'pattern']                                                            -- 查看所有表
SHOW TABLES FROM [表名]

SHOW CREATE TABLE [表名]                                                                -- 查看表结构
DESC 表名 / DESCRIBE 表名 / EXPLAIN 表名 / SHOW COLUMNS FROM 表名 [LIKE 'PATTERN']
SHOW TABLE STATUS [FROM db_name] [LIKE 'pattern']

ALTER TABLE [表名] [表的选项]                                                           -- 修改表
RENAME TABLE 原表名 TO [新表名]                                                         -- 对表进行重命名
RENAME TABLE 原表名 TO [库名.表名]                                                      -- 可将表移动到另一个数据库
ALTER TABLE [表名] [操作名]                                                             -- 修改表的字段结构
        操作名：
            ADD[ COLUMN] 字段定义                           -- 增加字段
                AFTER 字段名                                -- 增加在该字段名后⾯
                FIRST                                       -- 增加在第一个
            ADD PRIMARY KEY(字段名)                         -- 创建主键
            ADD UNIQUE [索引名] (字段名)                    -- 创建唯一索引
            ADD INDEX [索引名] (字段名)                     -- 创建普通索引
            DROP[COLUMN] 字段名                             -- 删除字段
            MODIFY[COLUMN] 字段名 字段属性                  -- 对字段属性进行修改，不能修改字段名(所有原有属性也需写上)
            CHANGE[COLUMN] 原字段名 新字段名 字段属性       -- 对字段名修改
            DROP PRIMARY KEY                                -- 删除主键(删除主键前需删除其AUTO_INCREMENT属性)
            DROP INDEX 索引名                               -- 删除索引
            DROP FOREIGN KEY 外键                           -- 删除外键

DROP TABLE[ IF EXISTS] 表名 ...                                                         -- 删除表
TRUNCATE [TABLE] 表名                                                                   -- 清空表数据
CREATE TABLE 表名 LIKE 要复制的表名                                                     -- 复制表结构
CREATE TABLE 表名 [AS] SELECT * FROM 要复制的表名                                       -- 复制表结构和数据
CHECK TABLE tbl_name [, tbl_name] ... [option] ...                                      -- 检查表是否有错误
OPTIMIZE [LOCAL | NO_WRITE_TO_BINLOG] TABLE tbl_name [, tbl_name] ...                   -- 优化表
REPAIR [LOCAL | NO_WRITE_TO_BINLOG] TABLE tbl_name [, tbl_name] ... [QUICK] [EXTENDED] [USE_FRM]    -- 修复表
ANALYZE [LOCAL | NO_WRITE_TO_BINLOG] TABLE tbl_name [, tbl_name] ...                    -- 分析表
```

### 数据操作

```
-- 增
INSERT [INTO] 表名 [(字段列表)] VALUES (值列表)[, (值列表), ...]
INSERT [INTO] 表名 SET 字段名=值[, 字段名=值, ...]
-- 查
SELECT 字段列表 FROM 表名
-- 删
DELETE FROM 表名
-- 改
UPDATE 表名 SET 字段名=新值[, 字段名=新值]
```

## 数据类型

### 整型
> `int(M)` M表⽰总位数
- 默认存在符号位，`unsigned` 属性修改
- 显⽰宽度，如果某个数不够定义字段时设置的位数，则前⾯以0补填，`zerofill` 属性修改
    例：int(5) 插⼊⼀个数'123'，补填后为'00123'
- 在满⾜要求的情况下，越⼩越好。
- 1表⽰bool值真，0表⽰bool值假。MySQL没有布尔类型，通过整型0和1表⽰。常⽤`tinyint(1)`表⽰布尔型。

|类型|字节|范围|
|---|---|---|
|tinyint    |1字节|   -128 ~ 127 ⽆符号位：0 ~ 255|
|smallint   |2字节|   -32768 ~ 32767|
|mediumint  |3字节|   -8388608 ~ 8388607|
|int        |4字节|
|bigint     |8字节|

### 浮点型
> 浮点型既⽀持符号位 `unsigned` 属性，也⽀持显⽰宽度 `zerofill` 属性。
>> 不同于整型，前后均会补填0.

> 定义浮点型时，需指定总位数和⼩数位数。
>> `float(M, D)` `double(M, D)`
>> M表⽰总位数，D表⽰⼩数位数。
>> M和D的⼤⼩会决定浮点数的范围。不同于整型的固定范围。
>> M既表⽰总位数（不包括⼩数点和正负号），也表⽰显⽰宽度（所有显⽰符号均包括）。
>> ⽀持科学计数法表⽰。
>> 浮点数表⽰近似值。

|类型|字节|范围|
|---|---|---|
|float(单精度)   |4字节
|double(双精度)  |8字节

### 定点数
> 保存⼀个精确的数值，不会发⽣数据的改变，不同于浮点数的四舍五⼊。
> 将浮点数转换为字符串来保存，每9位数字保存为4个字节。

|类型|说明|
|---|---|
|decimal|可变⻓度
|decimal(M, D) |M表⽰总位数，D表⽰⼩数位数


### 字符串类型
> varchar，第⼀个字节是空的，不存在任何数据，然后还需两个字节来存放字符串的⻓度，所以有效⻓度是65532个字节。

|类型|长度|说明|
|---|---|---|
|char|最多255个**字符**，与编码无关|定⻓字符串，速度快，但浪费空间|
|varchar|最多65535个**字符**，与编码有关|变⻓字符串，速度慢，但节省空间|
|blob||⼆进制字符串（tinyblob, blob, mediumblob, longblob）|
|text||⾮⼆进制字符串（tinytext, text, mediumtext, longtext）|
|binary||⽤于保存⼆进制字符串，对应char|
|varbinary||⽤于保存⼆进制字符串，对应varchar|

### 日期时间类型

|类型|字节|说明|举例|
|---|---|---|---|
|datetime    |8字节   |⽇期及时间  |1000-01-01 00:00:00 到 9999-12-31 23:59:59
|date        |3字节   |⽇期        |1000-01-01 到 9999-12-31
|timestamp   |4字节   |时间戳 1    |9700101000000 到 2038-01-19 03:14:07
|time        |3字节   |时间        |-838:59:59 到 838:59:59
|year        |1字节   |年份        |1901 - 2155

### 枚举和集合

|类型|格式|说明|
|---|---|---|
|枚举|enum(val1, val2, val3...)|以2个字节的整型(smallint)保存。每个枚举值，按保存的位置顺序，从1开始逐⼀递增。
|集合|set(val1, val2, val3...)|最多可以有64个不同的成员。以bigint存储，共8个字节。采取位运算的形式。