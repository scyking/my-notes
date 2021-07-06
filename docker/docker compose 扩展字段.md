# docker compose 扩展字段

标签（空格分隔）： docker

---

> [Extension fields](https://docs.docker.com/compose/compose-file/compose-file-v3/#extension-fields)

[toc]

## 概述
> 用于重用相同配置。

## 使用示例
> 定义必须以 `x-` 开头，然后以 `&` 开头的字符串为字段命名，之后就可以以 `*` 加上字段的名称进行引用。

### 扩展字段定义

```
version: "3.9"
x-custom:
  items:
    - a
    - b
  options:
    max-size: '12m'
  name: "custom"
```

### 扩展字段使用

- 重用配置

    ```
    logging:
      options:
        max-size: '12m'
        max-file: '5'
      driver: json-file
    ```

- 添加后效果

    ```
    version: "3.9"
    x-logging:
      &default-logging
      options:
        max-size: '12m'
        max-file: '5'
      driver: json-file
    
    services:
      web:
        image: myapp/web:latest
        logging: *default-logging
      db:
        image: mysql:latest
        logging: *default-logging
    ```

### 部分覆盖使用
> 使用`<<`进行配置。

```
version: "3.9"
x-volumes:
  &default-volume
  driver: foobar-storage

services:
  web:
    image: myapp/web:latest
    volumes: ["vol1", "vol2", "vol3"]
volumes:
  vol1: *default-volume
  vol2:
    << : *default-volume
    name: volume02
  vol3:
    << : *default-volume
    driver: default
    name: volume-local
```