# nginx 日志配置

标签（空格分隔）： nginx

---

[toc]

## 日志类型

- 访问日志（access_log），记录了用户IP地址，浏览器信息，请求的处理时间等信息。
- 错误日志（error_log），记录了访问出错的信息。

## 设置访问日志

### 语法

```
access_log path [format [buffer=size] [gzip[=level]] [flush=time] [if=condition]];  # 设置访问⽇志
access_log off;                                                                     # 关闭访问⽇志
```

- `path` 指定⽇志的存放位置。
- `format` 指定⽇志的格式。默认使⽤预定义的combined。
- `buffer` ⽤来指定⽇志写⼊时的缓存⼤⼩。默认是64k。
- `gzip` ⽇志写⼊前先进⾏压缩。压缩率可以指定，从1到9数值越⼤压缩⽐越⾼，同时压缩的速度也越慢。默认是1。
- `flush` 设置缓存的有效时间。如果超过flush指定的时间，缓存中的内容将被清空。
- `if` 条件判断。如果指定的条件计算为0或空字符串，那么该请求不会写⼊⽇志。

### 作用域

- `server`
- `location`
- `limit_except`

## 设置错误日志

### 语法

```
error_log file [level];
Default:
error_log logs/error.log error;
```

### 日志级别
> 只有⽇志的错误级别等于或⾼于level指定的值才会写⼊错误⽇志中。默认值是error。

- `debug`
- `info`
- `notice`
- `warn`
- `error`
- `crit`
- `alert`
- `emerg`

