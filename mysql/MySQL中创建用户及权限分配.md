# MySQL中创建用户及权限分配

标签（空格分隔）： mysql

---

> 需要`root`登录后操作。

## 创建用户

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

## 创建数据库

```
create database testdb DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
```

## 用户授权

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








