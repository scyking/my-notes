# github host配置

标签（空格分隔）： git

---

## 问题

> github 访问缓慢

## 解决方法

> 配置系统host文件

1. 配置 [github.com](https://github.com.ipaddress.com/)
1. 配置 [github.global.ssl.fastly.net](https://fastly.net.ipaddress.com/github.global.ssl.fastly.net#ipinfo)
1. cmd 下执行 `ipconfig /flushdns`（win10）

示例：
```
192.30.253.112 github.com 
199.232.5.194 github.global.ssl.fastly.net
```



