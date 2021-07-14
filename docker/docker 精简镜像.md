# docker 精简镜像

---

[toc]

## 概述

> Docker镜像由很多镜像层（Layers）组成，镜像层依赖于一系列的底层技术，比如文件系统（filesystems）、写时复制（copy-on-write）、联合挂载（union mounts）等。
> `Dockerfile` 中的每条指令都会创建一个镜像层，继而会增加整体镜像的尺寸。

- 精简镜像作用
  - 减少构建时间
  - 减少磁盘使用量
  - 减少下载时间
  - 因为包含文件少，攻击面减小，提高了安全性
  - 提高部署速度

## 方法

### 优化基础镜像

> 选用合适的、更小的基础镜像。例如，Linux系统镜像可使用 `Alpine`镜像。

- `alpine` 镜像
- `scratch` 镜像
- `busybox` 镜像

### 串联 `Dockerfile` 指令

> 把多个命令串联合并为一个 `RUN`（通过运算符 `&&` 和 `\` 来实现），减少构建镜像时产生的层数，以达到精简镜像的目的。

### 使用[多阶段构建](docker%20多阶段构建.md)

### 构建服务镜像技巧

> Docker在 `build` 镜像的时候，如果某个命令相关的内容没有变化，会使用上一次缓存（cache）的文件层。

- 使用技巧
  - 不变或者变化很少的体积较大的依赖库和经常修改的自有代码分开；
  - 因为cache缓存在运行Docker `build` 命令的本地机器上，建议固定使用某台机器来进行Docker `build`，以便利用cache。

### apt、apk工具使用技巧

1. 不使用安装建议性（非必须）的依赖。
    - 执行 `apt-get install -y` 时增加选项 `no-install-recommends`
    - 执行 `apk add` 时添加选项 `no-cache`
1. 组件的安装和清理串联在一条指令里，不在同一层是无法清理的。

## 问题

### 使用多阶段构建，工作路径改变

> 指定 `WORKDIR`，使用绝对路径操作文件。

### 使用 `scratch` 镜像问题

1. 缺少 `shell`

    > 使用 JSON 语法取代字符串语法。例如，将 `CMD ./hello` 替换为 `CMD ["./hello"]`。

1. 缺少调试工具，如 `ls`、`ps`、`ping` 等

    > 选择 `busybox` 或 `alpine` 镜像来替代 `scratch`。

1. 缺少 `libc`
    - 使用静态库
    - 拷贝库文件到镜像中
    - 使用 `busybox:glibc` 作为基础镜像
