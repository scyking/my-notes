# docker config 使用

标签（空格分隔）： docker

---

[toc]

## 概述

> 配置文件放入镜像中、设置环境变量、`volume`动态挂载等配置文件分发方式都降低了镜像的通用性。
> 使用`docker config`，无需将配置文件绑定到容器或使用环境变量来配置，保证了镜像的通用性。

- `docker config`特点
  - 安装在容器的文件系统中，而不是使用 `RAM` 磁盘
  - 可以随时添加或删除，服务可以共享一个配置
  - 可以与 `Environments` 或 `Labels` 结合使用，以获得最大的灵活性
  - config配置是不可变的，所以无法更改现有服务的文件，可以创建一个新的config来使用不同的文件
- 应用场景
  - nginx、redis等配置文件设定
  - 设定或更改资料库config等（通过指令增删config files）
  - 隐私资料需要使用`secret`来存
  - 需配合`swarm`服务使用

## 相关命令

|Command             |Description
|---|---|
|`docker config create` |Create a config from a file or STDIN
|`docker config inspect`|Display detailed information on one or more configs
|`docker config ls`     |List configs
|`docker config rm`     |Remove one or more configs

## docker-compose 中使用

> 需在3.3及以上版本中使用。

### 短语法

> 仅指定配置名称，安装在容器`/<config_name>`。

```yml
version: "3.9"
services:
  redis:
    image: redis:latest
    deploy:
      replicas: 1
    configs:
      - my_config
      - my_other_config
configs:
  my_config:
    file: ./my_config.txt
  my_other_config:
    external: true
```

### 长语法

- `source`，配置中定义的标识符
- `target`，配置文件在容器中位置，默认是`/<source>`。
- `uid`and`gid`，配置文件在容器中的`uid`或`gid`数值，默认是`0`（root）。
- `mode`，配置文件在容器中的权限，默认是`0444`。

```yml
version: "3.9"
services:
  redis:
    image: redis:latest
    deploy:
      replicas: 1
    configs:
      - source: my_config
        target: /redis_config
        uid: '103'
        gid: '103'
        mode: 0440
configs:
  my_config:
    file: ./my_config.txt
  my_other_config:
    external: true
```

## 示例

1. 创建配置

    ```bash
    echo "This is a config" | docker config create my-config -
    ```

1. 创建redis服务

    ```bash
    ## 创建redis服务，并指定配置my-config
    docker service create --name redis --config my-config redis:alpine
    ## 验证服务是否启动成功
    docker service ps redis
    ```

1. 查看redis服务中关联的配置信息

    ```bash
    ## 查看redis服务ID
    docker ps --filter name=redis -q
    ## 查看redis服务中关联的配置文件信息
    docker container exec $(docker ps --filter name=redis -q) ls -l /my-config
    docker container exec $(docker ps --filter name=redis -q) cat /my-config
    ```

1. 去除配置

    ```bash
    ## 去除服务与配置的关联
    docker service update --config-rm my-config redis
    ## 去除redis服务
    docker service rm redis
    ## 去除配置my-config
    docker config rm my-config
    ```
