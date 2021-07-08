# docker 搭建私有镜像仓库

标签（空格分隔）： docker

---

[toc]

## 安装部署

### 创建目录

```bash
mkdir /usr/local/docker
cd /usr/local/docker
```

### 编辑 `docker-compose.yml`

```yml
version: "3.7"
services:
  registry:
    restart: always
    image: registry
    container_name: registry
    ports:
      - 5000:5000
    volumes:
      - /usr/local/docker/registry-data:/var/lib/registry
```

### 启动容器

```bash
docker-compose up -d
```

### 验证是否启动成功

> 浏览器访问 `http://<host address>:5000/v2/`，正常情况会返回空的json对象。

## 测试使用

### 创建自己的镜像，为镜像打上tag

> 命名方式：`ip:port/image_name:version`

```bash
## 拉取镜像
docker pull ubuntu
## 打标
docker tag ubuntu 127.0.0.1:5000/ubuntu
```

### 配置镜像服务器地址

```bash
vi /etc/docker/daemon.json

## 内容如下：
{
    ## 镜像加速地址
    "registry-mirrors": [
            "<your acccelerate address>"
    ],
    ## 新增的镜像仓库地址
    "insecure-registries":  [
            "127.0.0.1:5000"
    ]
}

## 重启服务:
service docker restart

## 验证配置，看Insecure Registries节点是否多了一个配置。
docker info
```

### 推送镜像

```bash
docker push 127.0.0.1:5000/ubuntu
```

### 查看镜像

```bash
## 浏览器查看
http://127.0.0.1:5000/v2/_catalog
## 终端访问
curl -XGET http://127.0.0.1:5000/v2/_catalog
```

### 拉取镜像

```bash
## 删除本地镜像
docker rmi ubuntu 127.0.0.1:5000/ubuntu
## 拉取
docker docker 127.0.0.1:5000/ubuntu
```

## 安装图形界面

### 修改 `docker-compose.yml`

```yml
version: "3.7"
services:
  registry:
    restart: always
    image: registry
    container_name: registry
    ports:
      - 5000:5000
    volumes:
      - /usr/local/docker/registry-data:/var/lib/registry
      
  registry-ui:
    restart: always
    image: konradkleine/docker-registry-frontend:v2
    container_name: registry-ui
    environment:
      ENV_DOCKER_REGISTRY_HOST: 127.0.0.1
      ENV_DOCKER_REGISTRY_PORT: 5000
    ports:
      - 8080:80
    volumes:
      - /usr/local/docker/registry/server.crt:/etc/apache2/server.crt:ro
      - /usr/local/docker/registry-ui/server.key:/etc/apache2/server.key:ro
```

- `ENV_DOCKER_REGISTRY_HOST` : 镜像服务器地址
- `ENV_DOCKER_REGISTRY_PORT` : 镜像服务器端口

### 删除已存在容器

```bash
docker rm $(docker ps -aq)
```

### 重新启动

```bash
docker-compose up -d
```

### 访问

浏览器访问：

```text
http://127.0.0.1:8080/
```
