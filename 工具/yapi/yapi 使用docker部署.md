# yapi 使用docker部署

标签（空格分隔）： yapi docker

---

[toc]

## [参考地址](https://www.jianshu.com/p/a97d2efb23c5)

## 使用docker构建yapi

1. 启动mongodb

    ```bash
    docker run -d --name mongo-yapi mongo
    ```

1. 拉取Yapi镜像

    ```bash
    docker pull registry.cn-hangzhou.aliyuncs.com/anoyi/yapi
    ```

1. 自定义配置文件

    ```json
    {
      "port": "3000",
      "adminAccount": "admin@anoyi.com",
      "timeout":120000,
      "db": {
        "servername": "mongo",
        "DATABASE": "yapi",
        "port": 27017,
        "user": "anoyi",
        "pass": "anoyi.com",
        "authSource": "admin"
      }
    }
    ```

1. 初始化 Yapi 数据库索引及管理员账号

    ```bash
    docker run -it --rm \
      --link mongo-yapi:mongo \
      --entrypoint npm \
      --workdir /yapi/vendors \
      -v $PWD/config.json:/yapi/config.json \
      registry.cn-hangzhou.aliyuncs.com/anoyi/yapi \
      run install-server
    ```

1. 启动Yapi服务

    ```bash
    docker run -d \
      --name yapi \
      --link mongo-yapi:mongo \
      --workdir /yapi/vendors \
      -p 3000:3000 \
      -v $PWD/config.json:/yapi/config.json \
      registry.cn-hangzhou.aliyuncs.com/anoyi/yapi \
      server/app.js
    ```

## 使用Yapi

- 地址`http://localhost:3000`
- 账号`admin@admin.com`
- 密码`ymfe.org`

## 其他相关操作

- 关闭yapi`docker stop yapi`
- 启动yapi`docker start yapi`

## 手动构建 yapi 镜像

1. 下载 YAPI 到本地--[下载地址](https://github.com/YMFE/yapi/releases)
2. 编辑 Dockerfile

    ```bash
    FROM node:12-alpine as builder
    RUN apk add --no-cache git python make openssl tar gcc
    COPY yapi.tar.gz /home
    RUN cd /home && tar zxvf yapi.tar.gz && mkdir /api && mv /home/yapi-1.8.0 /api/vendors
    RUN cd /api/vendors && \
    npm install --production --registry https://registry.npm.taobao.org
    FROM node:12-alpine
    MAINTAINER 545544032@qq.com
    ENV TZ="Asia/Shanghai" HOME="/"
    WORKDIR ${HOME}
    COPY --from=builder /api/vendors /api/vendors
    COPY config.json /api/
    EXPOSE 3000
    ENTRYPOINT ["node"]
    ```

3. 构建镜像`docker build -t yapi .`
