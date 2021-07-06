# go 使用 c/c++ 

标签（空格分隔）： go

---

[toc]

## 概述
> 在 `go` 项目中使用 `c/c++`。

### 实现方式

- 直接嵌套在 `go` 文件中
- 导入动态库 `.so` 或 `dll` 
- 直接引用 `c/c++` 文件

### 环境支持

- Linux 具备 `gcc` 与 `g++` 即可
- Windows 需要安装 `mingw`，否则编译时会有这类错：`cannot find -lmingwex`

## 直接嵌套

- 但凡要引用与 `c/c++` 相关的内容，写到 `go` 文件的头部**注释**里面
- 嵌套的 `c/c++` 代码必须符合其语法，不与 `go` 一样
- `import "C"` 这句话要紧随**注释**后，不要换行，否则报错
- `go` 代码中调用 `c/c++` 的格式是: `C.xxx()`。例如 `C.add(2, 1)`

```
package main
/*
// C 标志io头文件，你也可以使用里面提供的函数
#include <stdio.h> 

void pri(){
	printf("hey");
}

int add(int a,int b){
	return a+b;
}
*/
import "C"  // 切勿换行再写这个

import "fmt"

func main() {
	fmt.Println(C.add(2, 1))
}
```

## 导入动态库

### 优缺点

- 动态库破解十分困难，如果你的 go 代码泄露，核心动态库没那么容易被攻破
- 动态库会在被使用的时候被加载，影响速度
- 操作难度比方式一麻烦不少

### 使用说明
> 如果动态库不存在，将会报**找不到定义**之类的错误信息

- `CFLAGS: -I路径` ：指明头文件所在路径。例如，`-Iinclude` 指明 当前项目根目录的 `include` 文件夹
- `LDFLAGS: -L路径 -l名字`：指明动态库的所在路径。例如，`-Llib -llibvideo`，指明在 `lib` 下面以及它的名字 `video`

### 示例

1. 假设项目目录如下

    ```
    |-project
    |  |-lib
    |  |  |-libvideo.dll
    |  |  |-libvideo.so
    |  |-include
    |  |  |-video.h
    |  |-src
    |  |  |-main.go
    ```

1. 头文件 `.h` 如下

    ```
    //video.h
    #ifndef VIDEO_H
    #define VIDEO_H
    // 声明
    void exeFFmpegCmd(char* cmd); 
    #endif
    ```

1. 源文件 `.c` 如下

    ```
    #include <stdio.h>
    #include "video.h"
    
    // 实现
    void exeFFmpegCmd(char* cmd){ 
        // ....
        printf("finish");
    }
    ```

1. 生成动态库
> 使用 `gcc`/`g++` 生成 `.so`库，或 win 下生成 `dll`。

    ```
    gcc video.c -fPIC -shared -o libvideo.so
    ```

1. `main.go`
> 将动态库放到指定目录或者当前项目里，再引用。

    ```
    package main
    
    /*
    
    #cgo CFLAGS: -Iinclude
    
    #cgo LDFLAGS: -Llib -llibvideo
    
    #include "video.h"
    
    */
    import "C"
    
    import "fmt"
    
    func main() {
        cmd := C.CString("ffmpeg -i ./xxx/*.png ./xxx/yyy.mp4")
        C.exeFFmpegCmd(&cmd)
    }
    ```

## 直接引用

1. 假设项目目录如下

    ```
    |-util
    |  |-util.h
    |  |-util.c
    |  |-util.go
    ```

1. `util.h`

    ```
    int sum(int a,int b);
    ```

1. `util.c`

    ```
    #include "util.h"
    int sum(int a,int b){
        return (a+b);
    }
    ```

1. `util.go`

    ```
    package util
    
    /*
    #include "util.c"
    */
    import "C"
    
    import "fmt"
    
    func GoSum(a,b int) int {
        s := C.sum(C.int(a),C.int(b))
        fmt.Println(s)
    }
    ```

1. `main.go`

    ```
    package main
    
    func main(){
        util.GoSum(4,5)
    }
    ```