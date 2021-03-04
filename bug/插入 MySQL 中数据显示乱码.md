# 插入 MySQL 中数据显示乱码 

标签（空格分隔）： bug

---

## 问题描述

已做以下处理：

- 使用 `IDEA` 进行开发，并设置字符编码为 `UTF-8` （即在 `File Ecodings` 设置中将 `Global Encoding`、`Project Encoding` 及 `Properties Files` 编码均设置为 `UTF-8` ）
- MySQL 数据库、表编码设置为'UTF-8'，未对字段设置特殊编码

通过 `DeBug` 发现如下现象：

- 从前台获取到的数据，在存入数据库之前均未乱码
- 直接操作数据库（手写sql语句），插入的数据未乱码
- 后台从数据库中取出的数据未乱码

## 解决方案

将后台中连接数据库配置修改为：

```
...
spring.datasource.mysql.url=jdbc:mysql://127.0.0.1:3306/mysql?useUnicode=true&characterEncoding=utf8
...
```






