# go module 使用

---

[toc]

> [官方文档](https://github.com/golang/go/wiki/Modules)

## 概述

> go module 是 go 官方自带的 go 依赖管理库。（在1.13版本正式推荐使用）

## 开启 go module

```bash
# windows
go env -w GO111MODULE=on
# linux
export GO111MODULE=on
```

## 初始化模块

> 进入项目所在目录，执行如下命令会生成 `go.mod` 文件。

```bash
go mod init <模块名>
```

## 依赖管理

### 依赖整理

> 执行如下命令，会将缺失依赖写入 `go.mod` 文件中并移除无用依赖。

```bash
go mod tidy
```

### 下载依赖

> 执行如下命令，会将依赖下载至本地 `GOPATH` 目录下，并生成 `go.sum` 文件。

```bash
go mod download
```

### 导入依赖

> 执行如下命令，会将 `GOPATH` 目录中依赖复制到模块根目录下的 `vendor` 文件夹下。
> 建议一些使用了被墙包的话可以这么处理，方便用户快速使用命令 `go build -mod=vendor` 编译。

```bash
go mod vendor
```

### 其他依赖管理命令

```bash
go list -m                      # 列出与 Modules 相关信息
go list -m all                  # 列出项目中的所有依赖及其依赖树
go list -m -versions <包名>     # 列出指定包的所有版本号
```

## 其他 `go mod` 命令

```bash
go mod edit             # 手动修改依赖文件
go mod graph            # 打印依赖图
go mod verify           # 校验依赖
go mod why <pkg>        # 解释依赖某个软件包原因
```
