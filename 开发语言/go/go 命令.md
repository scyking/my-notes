# go 命令

标签（空格分隔）： go

---

[toc]

## 概览

```
Go is a tool for managing Go source code.

Usage:

        go <command> [arguments]

The commands are:

        bug         start a bug report
        build       compile packages and dependencies
        clean       remove object files and cached files
        doc         show documentation for package or symbol
        env         print Go environment information
        fix         update packages to use new APIs
        fmt         gofmt (reformat) package sources
        generate    generate Go files by processing source
        get         download and install packages and dependencies
        install     compile and install packages and dependencies
        list        list packages or modules
        mod         module maintenance
        run         compile and run Go program
        test        test packages
        tool        run specified go tool
        version     print Go version
        vet         report likely mistakes in packages

Use "go help <command>" for more information about a command.
```

## 常用命令

### go build
> go语言编译命令

- 无参数编译 `go build`
- 多文件编译 `go build file1.go file2.go ...`
- 根绝包编译 `go build -o main <包路径>`

|附加参数	    |备  注
|---|---|
|`-v`	        |编译时显示包名
|`-p n`	        |开启并发编译，默认情况下该值为 CPU 逻辑核数
|`-a`	        |强制重新构建
|`-n`	        |打印编译时会用到的所有命令，但不真正执行
|`-x`	        |打印编译时会用到的所有命令
|`-race`	    |开启竞态检测
|`-o`           |指定输出文件，后接要编译的包名

### go run
> `go run`命令会编译源码，并且直接执行源码的 `main()` 函数，不会在当前目录留下可执行文件。

### go get
> `go get` 命令可以借助代码管理工具通过远程拉取或更新代码包及其依赖包，并自动完成编译和安装。

|附加参数	    |备  注
|---|---|
|`-d`           |只下载不安装
|`-f`           |只有在你包含了 -u 参数的时候才有效，不让 -u 去验证 import 中的每一个都已经获取了，这对于本地 fork 的包特别有用
|`-fix`         |在获取源码之后先运行 fix，然后再去做其他的事情
|`-t`           |同时也下载需要为运行测试所需要的包
|`-u`           |强制使用网络去更新包和它的依赖包
|`-v`           |显示执行的命令

### go test
> `go test` 命令，会自动读取源码目录下面名为 *_test.go 的文件，生成并运行测试用的可执行文件。
> `go test` 命令默认情况下会自动测试以`_test`后缀结尾文件中以`Test`前缀开头方法。

1. 常用参数
    - `-bench regexp` 执行相应的 benchmarks，例如 -bench=.；
    - `-cover` 开启测试覆盖率；
    - `-run regexp` 只运行 regexp 匹配的函数，例如 -run=Array 那么就执行包含有 Array 开头的函数；
    - `-v` 显示测试的详细命令。
1. `testing.T`提供的日志输出

    |方  法	|备  注
    |---|---|
    |`Log`      |打印日志，同时结束测试
    |`Logf`	    |格式化打印日志，同时结束测试
    |`Error`	|打印错误日志，同时结束测试
    |`Errorf`	|格式化打印错误日志，同时结束测试
    |`Fatal`	|打印致命日志，同时结束测试
    |`Fatalf`	|格式化打印致命日志，同时结束测试