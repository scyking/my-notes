# mysql 使用

标签：mysql

---

[toc]

## Windows 上安装 mysql 8.x

### 安装步骤（`zip`）

- [下载 mysql](https://dev.mysql.com/downloads/mysql/)
- 配置环境变量
    - 添加新系统变量（`MYSQL_HOME` 变量值为zip解压目录）
    - 在path里添加 `%MYSQL_HOME%\bin;`
- 初始化([详细介绍](https://dev.mysql.com/doc/refman/5.7/en/data-directory-initialization-mysqld.html))
进入解压后bin目录下，在cmd下执行【5.6版本及以下包含data目录，不需要执行】
```
mysqld --initialize --console
```
- 记录随机产生的密码（用于首次登陆）
- 安装服务 
```
mysqld --install 服务名 --服务名可省略
```
- 启动服务
```
net start mysql 
```

### 删除mysql

- 开始 --> 控制面板 --> 管理工具 --> 服务
- 停掉mysql服务
- 卸载mysql
```
sc delete mysql
```
- 删除注册表
    - 执行regedit，打开注册表
    - 删除如下包含mysql相关的注册表(PS:删除MySQL/MySQLD)
```
HKEY_LOCAL_MACHINE/SYSTEM/ControlSet001/Services/Eventlog/Application/MySQL
HKEY_LOCAL_MACHINE/SYSTEM/ControlSet002/Services/Eventlog/Application/MySQL
HKEY_LOCAL_MACHINE/SYSTEM/CurrentControlSet/Services/Eventlog/Application/MySQL
```

### 问题解决

- 不是有效的win32程序
删除bin目录下名为mysqld的文件（大小为0k）。

- Navicat for MySQL 1251错误（不支持身份认证方式）
修改加密规则
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

- 报 `Please read "Security" section of the manual to find out how to run mysqld as root!
` 错误

    1. 最简单的方式是在执行命令后添加`--user=root`参数。 （强制使用系统`root`用户初始化，不推荐）
    1. 创建系统新用户，执行命令。
    
- 报 `Can't open the mysql.plugin table. Please run mysql_upgrade to create it.` 错误

    进入mysql安装目录，执行 `mysql_install_db` 命令。
    
## 创建用户及权限分配

### 创建用户

- 允许本地 IP 访问 `localhost`, `127.0.0.1`

```
create user 'test'@'localhost' identified by '123456';
```

- 允许外网 IP 访问

```
create user 'test'@'%' identified by '123456';
```

- 修改用户密码

```
update table mysql.user set password=password('new_password') where User='test' and Host='localhost';
```

- 刷新授权

```
flush privileges;
```

### 创建数据库

```
create database testdb DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
```

### 用户授权

- 授予用户通过外网IP对于该数据库的全部权限

```
grant all privileges on `testdb`.* to 'test'@'%' identified by '123456';
```

- 授予用户在本地服务器对该数据库的全部权限

```
grant all privileges on `testdb`.* to 'test'@'localhost' identified by '123456';
```

- 刷新授权

```
flush privileges;
```

    
    
    