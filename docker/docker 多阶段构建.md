# docker 多阶段构建

---

[toc]

## 概述

> 构建编译型语言镜像，需要先在一个 `Dockerfile` 中编译，然后再使用另外一个 `Dockerfile` 把编译好的文件放到镜像中。这样维护两个 `Dockerfile` 并不理想。
> 使用多阶段构建，可以在 `Dockerfile` 中使用多个 `FROM` 指令。每个 `FROM` 指令可以使用不同的基础，并且每个指令都开始一个新的构建。可以选择性地将工件从一个阶段复制到另一个阶段，从而在最终 `image` 中只留下想要的内容。

### 旧版本仅支持单个 `FROM` 指令原因

> 在 `17.05` 版本之前，只允许 `Dockerfile` 中出现一个 `FROM` 指令。
> Docker镜像的每一层只记录文件变更，在容器启动时，Docker会将镜像的各个层进行计算，最后生成一个文件系统，这个被称为 **联合挂载**。
> Docker的各个层是有相关性的，在联合挂载的过程中，系统需要知道在什么样的基础上再增加新的文件。那么这就要求一个Docker镜像只能有一个起始层，只能有一个根。所以，`Dockerfile` 中，就只允许一个 `FROM` 指令。

### 多个 `FROM` 指令的意义

> 多个 `FROM` 指令并不是为了生成多根的层关系，最后生成的镜像，仍以最后一条 `FROM` 为准，之前的 `FROM` 会被抛弃。

- 使用场景
  - 编译环境与运行环境分离

## 示例

### 多阶段构建出现之前

> 运行 `build.sh` 脚本，构建第一个 `image`，从中创建容器以复制工件，然后构建第二个 `image`。

- `Dockerfile.build`

    ```dockerfile
    FROM golang:1.7.3
    WORKDIR /go/src/github.com/alexellis/href-counter/
    COPY app.go .
    RUN go get -d -v golang.org/x/net/html \
    && CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .
    ```

- `Dockerfile`

    ```dockerfile
    FROM alpine:latest  
    RUN apk --no-cache add ca-certificates
    WORKDIR /root/
    COPY app .
    CMD ["./app"]
    ```

- `build.sh`

    ```sh
    #!/bin/sh

    echo Building alexellis2/href-counter:build
    docker build --build-arg https_proxy=$https_proxy --build-arg http_proxy=$http_proxy \  
    -t alexellis2/href-counter:build . -f Dockerfile.build
    docker container create --name extract alexellis2/href-counter:build  
    docker container cp extract:/go/src/github.com/alexellis/href-counter/app ./app  
    docker container rm -f extract
    echo Building alexellis2/href-counter:latest
    docker build --no-cache -t alexellis2/href-counter:latest .
    rm ./app
    ```

### 使用多阶段构建

- `Dockerfile`

    ```dockerfile
    # 编译阶段
    FROM golang:1.7.3
    WORKDIR /go/src/github.com/alexellis/href-counter/
    RUN go get -d -v golang.org/x/net/html  
    COPY app.go .
    RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .

    # 运行阶段
    FROM alpine:latest  
    RUN apk --no-cache add ca-certificates
    WORKDIR /root/
    COPY --from=0 /go/src/github.com/alexellis/href-counter/app .
    CMD ["./app"]  
    ```

- 执行

    ```bash
    docker build -t app:latest .
    ```

## 为多构建阶段命名

- 默认情况下，阶段未命名。可通过整数来引用 `FROM` 指令，从第 0 个开始。

- 通过向 `FROM` 指令添加 `as <name>` 来命名阶段。

    ```dockerfile
    FROM golang:1.7.3 as builder
    WORKDIR /go/src/github.com/alexellis/href-counter/
    RUN go get -d -v golang.org/x/net/html  
    COPY app.go    .
    RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .

    FROM alpine:latest  
    RUN apk --no-cache add ca-certificates
    WORKDIR /root/
    COPY --from=builder /go/src/github.com/alexellis/href-counter/app .
    CMD ["./app"] 
    ```

## 停在特定的构建阶段

> 构建镜像时，不一定需要构建 `Dockerfile` 每个阶段，可以指定目标构建阶段。

```bash
# 在名为 builder 的阶段停止
docker build --target builder -t alexellis2/href-counter:latest .
```

- 适用场景
  - 调试特定的构建阶段
  - 在debug阶段，启用所有调试或工具，而在production阶段尽量精简
  - 在testing阶段，填充测试数据，但在production阶段则使用生产数据

## 使用外部镜像作为stage

> 使用多阶段构建时，不仅可以从 `Dockerfile` 中创建的镜像中进行复制，还可以使用 `COPY –from` 指令从单独的image中复制（本地image名称，本地/Docker注册表中可用的标记或标记ID）。

```dockerfile
COPY --from=nginx:latest /etc/nginx/nginx.conf /nginx.conf
```
