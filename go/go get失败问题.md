# go get 失败问题

标签（空格分隔）： go

---

## 问题描述
> 使用`go get`安装第三方包，会出现失败的情况。
> 配置代理也不能解决。

```
$ go get collidermain
package golang.org/x/net/websocket: unrecognized import path 
"golang.org/x/net/websocket" (https fetch: Get https://golang.org/x/net/websocket?go-get=1: 
dial tcp 216.239.37.1:443: i/o timeout)
```

## 解决方法
> golang 在 github 上建立了一个镜像库，如 `https://github.com/golang/net` 即是 `https://golang.org/x/net` 的镜像库获取 `golang.org/x/net` 包。

1. 在`GOPATH`下创建`/src/golang.org/x`目录
2. 在目录下执行`git clone https://github.com/golang/net.git`或者直接拷贝源码到该目录

PS：根据报错信息，在[镜像库](https://github.com/golang/)中拉取源码。如`sys`、`tools`等