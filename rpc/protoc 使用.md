# protoc 使用

标签（空格分隔）： rpc

---

[toc]

## 概述
> 用于编译`.proto`文件，生成源代码文件。

## windows 下使用

### [下载最新版本](https://github.com/protocolbuffers/protobuf/releases/)

### 配置环境变量

1. 解压下载的压缩包
2. 将`protoc.exe`文件所在目录（一般是`%gopath%/bin`），添加到环境变量`Path`中

### 常用参数

- `-I`，`--proto_path`      指定了`import`要导入的和要编译的`.proto`文件所在目录，可指定多个
- `--version`               显示版本信息
- `-h`, `--help`            显示帮助信息 
- `--cpp_out=OUT_DIR`       C++ 
- `--csharp_out=OUT_DIR`    C# 
- `--java_out=OUT_DIR`      Java 
- `--js_out=OUT_DIR`        JavaScript 
- `--objc_out=OUT_DIR`      Objective C 
- `--php_out=OUT_DIR`       PHP 
- `--python_out=OUT_DIR`    Python 
- `--ruby_out=OUT_DIR`      Ruby 

### 支持 golang

1. 安装编译插件

    ```
    go install google.golang.org/protobuf/cmd/protoc-gen-go
    ```

1. 将生成的`%gopath%/bin/protobuf-gen-go.exe`可执行文件，放入`protoc.exe`文件所在目录

1. 使用命令

    ```
    ## `*`表示文件名称 
    protoc --go_out=. *.proto

    protoc --go_out=plugins=grpc:. *.proto
    ```
    
## 问题

### go protoc编译问题 `-protoc-gen-go: unable to determine Go import path for ***.proto`

> 在`.proto`文件中，添加`option go_package=`参数。

### go protoc编译问题 `--go_out: protoc-gen-go: plugins are not supported; use 'protoc --go-grpc_out=...' to generate gRPC`

1. 安装`protoc-gen-go-grpc`插件

    ```
    // 如未下载过，使用`go get`
    go install google.golang.org/grpc/cmd/protoc-gen-go-grpc
    ```

1. 将生成的`protoc-gen-go-grpc.exe`可执行文件，放入`protoc.exe`文件所在目录

1. 修改执行命令

    ```
    // 将执行命令
    protoc --go_out=plugins=grpc:. *.proto
    // 修改为
    protoc --go-grpc_out=. *.proto
    ```
    
### 加`--go-grpc_out`参数，执行后未生成`*_grpc.pb.go`文件

> `.proto`文件中需定义`service`方法，才可正常生成。