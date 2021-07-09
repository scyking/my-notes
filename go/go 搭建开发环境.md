# go 搭建开发环境

[toc]

> [Download and install](https://golang.google.cn/doc/install)

## [下载 Go](https://golang.google.cn/dl/)

> Linux 可使用 `wget` 命令，直接下载。

```bash
## 进入安装包要下载的位置
cd /usr/local/
## 下载
wget https://dl.google.com/go/go1.16.5.linux-amd64.tar.gz
```

## 安装 Go

### Windows

1. 打开下载的 Go `MSI` 文件，直接安装。
1. CMD 下使用 `go version` 验证是否安装成功。

### Linux

1. 解压
`tar -vxf go1.16.5.linux-amd64.tar.gz`
1. 配置环境变量
`echo "export PATH=$PATH:/usr/local/go/bin" >> /etc/profile`
1. 生效配置
`source /etc/profile`
1. 查看版本
`go version`
1. 查看安装位置
`which go`

## 环境变量

- 使用命令 `go env` 查看当前环境变量配置。
- 使用命令 `go env -w [变量名]=[变量值]` 修改环境变量。

### `GO111MODULE`

> Go 1.11 版本推出`MODULE`模式，以逐步取代 `GOPATH` 模式。`GOPATH` 模式，源码必须放到`%GOPATH%/src` 目录下且只能有一个 `main()` 方法，不够灵活。`GO111MODULE` 配置参数如下：

- `on`
    > go 会忽略 `$GOPATH`和 `vendor`文件夹，只根据`go.mod`下载依赖。
- `off`
    > 不使用新特性 `Go Modules` 支持，它会查找 `vendor` 目录和 `$GOPATH` 来查找依赖关系，也就是继续使用`GOPATH`模式。
- `auto`或未设置
    > 则根据当前项目目录下是否存在 `go.mod` 文件或 `$GOPATH/src` 之外并且其本身包含 `go.mod` 文件时才会使用新特性 `Go Modules` 模式。

### `GOPATH`

> Go 语言的工作空间。

### `GOROOT`

> Go 语言编译、工具、标准库等的安装路径。

### `GOPROXY`

> Go 语言代理地址。为提高安装模块速度，使用如下代理配置。

```bash
## Windows
go env -w GOPROXY="https://goproxy.io,direct"

## Linux
export GOPROXY="https://goproxy.io,direct"
```

## 多文件执行

1. 使用`go run + [go文件]`

    ```bash
    go run *.go
    go run [go文件...]
    ```

1. 使用 `go build`，执行生成的可执行文件。
