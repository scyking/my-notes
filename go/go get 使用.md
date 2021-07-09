# go get 使用

标签（空格分隔）： go

---

[toc]

## 概述

> 用于远程拉取或更新代码包及其依赖包，并自动完成编译和安装。

### 参数

```bash
-d          # 只下载不安装。
-f          # 仅在使用 -u 参数时生效。让命令程序忽略掉对已下载代码包的导入路径的检查。
-fix        # 让命令程序在下载代码包后先执行修正动作，而后再进行编译和安装。
-t          # 同时也下载需要为运行测试所需要的包。
-u          # 下载丢失的包，但不会更新已经存在的包。
-v          # 显示操作流程的日志及信息，方便检查错误。
```

## 远程包的路径格式

> 路径共由 3 个部分组成，例如 `github.com/golang/go`，分别是网站域名、作者（机构）、项目名。

- 网站域名：表示代码托管的网站，类似于电子邮件 `@` 后面的服务器地址。
- 作者或机构：表明这个项目的归属，一般为网站的用户名，如果需要找到这个作者下的所有项目，可以直接在网站上通过搜索“域名/作者”进行查看。这部分类似于电子邮件 `@` 前面的部分。
- 项目名：每个网站下的作者或机构可能会同时拥有很多的项目，图中标示的部分表示项目名称。

## 问题

### 拉取安装包超时问题

> 使用`go get`安装第三方包失败，并且配置代理也不能解决。

- 错误信息

    ```go
    $ go get collidermain
    package golang.org/x/net/websocket: unrecognized import path 
    "golang.org/x/net/websocket" (https fetch: Get https://golang.org/x/net/websocket?go-get=1: 
    dial tcp 216.239.37.1:443: i/o timeout)
    ```

- 解决方法
    > golang 在 github 上建立了一个镜像库，如 `https://github.com/golang/net` 即是 `https://golang.org/x/net` 的镜像库获取 `golang.org/x/net` 包。

    1. 在`GOPATH`下创建`/src/golang.org/x`目录
    1. 在目录下执行`git clone https://github.com/golang/net.git`或者直接拷贝源码到该目录

PS：根据报错信息，在[镜像库](https://github.com/golang/)中拉取源码。如`sys`、`tools`等
