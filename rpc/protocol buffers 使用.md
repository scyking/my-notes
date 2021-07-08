# protocol buffers 使用

标签（空格分隔）： rpc

---

[toc]

> 详见 [Protocol Buffers](https://developers.google.cn/protocol-buffers) 官方文档。

## 概述
> Protocol Buffers 是 google 推出的一种数据序列化格式，简称protobuf。

### 优点

- 支持多种编程语言
- 序列化数据体积小
- 反序列化速度快
- 序列化和反序列化代码自动生成

### 缺点

- 可读性差，缺乏自描述性

## 使用步骤

1. 定义消息格式，编写`.proto`文件
2. 选择合适的protobuf实现框架，对`.proto`文件进行编译，生成对应的源代码文件
3. 在代码中调用生成的源代码接口，完成序列化和反序列化功能

### 定义消息格式
> 在`.proto`文件中定义消息格式。`addressbook.proto`文件内容如下：

```
syntax = "proto3";
package tutorial;

import "google/protobuf/timestamp.proto";

option go_package = "github.com/protocolbuffers/protobuf/examples/go/tutorialpb";

message Person {
  string name = 1;
  int32 id = 2;  // Unique ID number for this person.
  string email = 3;

  enum PhoneType {
    MOBILE = 0;
    HOME = 1;
    WORK = 2;
  }

  message PhoneNumber {
    string number = 1;
    PhoneType type = 2;
  }

  repeated PhoneNumber phones = 4;

  google.protobuf.Timestamp last_updated = 5;
}

// Our address book file is just one of these.
message AddressBook {
  repeated Person people = 1;
}
```

### 编译`.proto`文件
> 编译`.proto`文件，生成`.pb.go`源代码文件。（Go）

1. 安装编译插件

    ```
    go install google.golang.org/protobuf/cmd/protoc-gen-go
    ```
2. 编译

    ```
    protoc -I=$SRC_DIR --go_out=$DST_DIR $SRC_DIR/addressbook.proto
    ```
    
    - `-I` 等价于 `--proto_path`：指定了`import`要导入的和要编译的`.proto`文件所在目录，可指定多个
    - `--go_out` 输出目标文件路径
    - `$SRC_DIR/addressbook.proto` 指定需要编译的 `.proto` 文件

### 序列化

```
book := &pb.AddressBook{}
// ...

// Write the new address book back to disk.
out, err := proto.Marshal(book)
if err != nil {
        log.Fatalln("Failed to encode address book:", err)
}
if err := ioutil.WriteFile(fname, out, 0644); err != nil {
        log.Fatalln("Failed to write address book:", err)
}
```

### 反序列化

```
// Read the existing address book.
in, err := ioutil.ReadFile(fname)
if err != nil {
        log.Fatalln("Error reading file:", err)
}
book := &pb.AddressBook{}
if err := proto.Unmarshal(in, book); err != nil {
        log.Fatalln("Failed to parse address book:", err)
}
```

## proto3 与 proto2 区别
> 优先使用`proto3`，`proto3`去掉了一些复杂的语法和特性，更强调约定而弱化语法。

- 在第一行非空白非注释行，必须写：`syntax = "proto3";`
- 字段规则移除了 `required`，并把 `optional` 更名为 `singular`
- `repeated`字段默认采用 `packed` 
- 增加 Go、Ruby、JavaNano 语言支持
- 移除了 `default` 选项，默认值使用字段类型自身默认值，以节省空间
    - `string`, 默认值是空字符串（`empty string`）
    - `bytes`, 默认值是空`bytes`（e`mpty bytes`）
    - `bool`, 默认值是`false`
    - `numeric`, 默认值是`0`
    - `enum`, 默认值是第一个枚举值（value必须为`0`）
    - `repeated`，默认值为`empty`，通常是一个空`list`
- 枚举类型的第一个字段必须为 `0`
- 移除了对分组的支持，使用嵌套代替
- 移除了对扩展的支持，新增了 `Any` 类型
- 增长了 `JSON` 映射特性