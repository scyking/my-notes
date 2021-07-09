# sublime 配置 go 开发环境

标签（空格分隔）： go sublime

---

[toc]

## 安装go语言

- [windows 下载地址](https://golang.google.cn/dl/)

## sublime 安装 Package Control（插件管理工具）

- `ctrl` + `` `，调起控制台
- 输入安装代码，回车

    ```bash
    import urllib.request,os,hashlib; h = '6f4c264a24d933ce70df5dedcf1dcaee' + 'ebe013ee18cced0ef93d5f746d80ef60'; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); by = urllib.request.urlopen( 'http://packagecontrol.io/' + pf.replace(' ', '%20')).read(); dh = hashlib.sha256(by).hexdigest(); print('Error validating download (got %s instead of %s), please try manual install' % (dh, h)) if dh != h else open(os.path.join( ipp, pf), 'wb' ).write(by)
    ```

## sublime 安装 GoSublime 插件

- 依次点击`Preferences` --> `Browse Packages`，进入sublime的package目录
- 运行 `git clone https://margo.sh/GoSublime` ，将代码下载到package目录
- 重启sublime（未安装go语言环境会报错）
- 使用 `Ctrl`+`Shift`+`p` 调起 Package Control
- 选中  `default Settings`，回车。（会打开gosublime的配置文件）
- 修改配置文件

    ```bash
    "gscomplete_enabled": true,
     // Whether or not gsfmt is enabled
    "fmt_enabled": true,
    ```

## go 环境变量

> 使用`go env`查看

### 设置环境变量

> Go version >= 1.13可直接使用命令

```bash
go env -w [变量名]=[变量值]
```

### `GOPATH`

> go的工作目录（workspace）

### `GOROOT`

> go语言编译、工具、标准库等的安装路径

### `GO111MODULE`

> 是否开启GoModule特性
> go1.11时推出

- `on`：go 会忽略 `$GOPATH`和 `vendor`文件夹,只根据`go.mod`下载依赖
- `off`：不适用新特性 `Go Modules` 支持，它会查找 `vendor` 目录和 `$GOPATH` 来查找依赖关系，也就是继续使用`GOPATH`模式
- `auto`或未设置：则根据当前项目目录下是否存在 `go.mod` 文件或 `$GOPATH/src` 之外并且其本身包含 `go.mod` 文件时才会使用新特性 `Go Modules` 模式

#### go.mod 相关命令

- `go mod help` 查看帮助
- `go mod init<项目模块名称>` 初始化模块，会在项目根目录下生成 go.mod 文件
- `go mod tidy` 根据 `go.mod` 文件来处理依赖关系
- `go mod vendor` 将依赖包复制到项目下的 vendor 目录。建议一些使用了被墙包的话可以这么处理，方便用户快速使用命令 `go build -mod=vendor` 编译
- `go list -m all` 显示依赖关系。
- `go list -m -json all` 显示详细依赖关系。
- `go mod download` 下载依赖。参数是非必写的，`path` 是包的路径，`version` 是包的版本。

## go 代理配置

```bash
go env -w GOPROXY="https://goproxy.io,direct"
```

## go 多文件执行

### `GO111MODULE=off` 配置下

> 只能有一个`main`方法

1. 使用`go run`+所有文件

    ```bash
    go run *.go
    // or
    go run [go文件...]
    ```

1. 使用`go build`

    ```bash
    go build
    [exe文件]
    ```
