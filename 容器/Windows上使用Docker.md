# Windows上使用Docker

标签：docker

---

[TOC]

## 基本概念

- `image`：镜像，相当于是一个root文件系统。
- `container`：容器，就像是面向对象程序设计中的实例和类一样。镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。
- `repository`：仓库，可看成一个代码控制中心，用来保存镜像。

## 安装

- 下载和安装 Docker Toolbox

    [下载地址（aliyun）](http://mirrors.aliyun.com/docker-toolbox/windows/docker-toolbox/)
    
- 配置

1. 将镜像安装到其他盘符（一般默认是c盘）。
    - notepad .bash_profile 创建和打开.bash_profile 配置文件
    - 添加export MACHINE_STORAGE_PATH='E:\docker'
    - 在E盘创建名为docker的文件夹，在其下创建名为cache的文件夹，将安装文件下的boot2docker.iso拷贝到该文件夹

1. 使用[阿里云镜像提速](https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors)。
```
docker-machine create --engine-registry-mirror=https://czx8lu07.mirror.aliyuncs.com -d virtualbox default
```
    
1. 安装docker-machine。（需先安装docker）

安装命令：
```
$ base=https://github.com/docker/machine/releases/download/v0.16.0 &&
  mkdir -p "$HOME/bin" &&
  curl -L $base/docker-machine-Windows-x86_64.exe > "$HOME/bin/docker-machine.exe" &&
  chmod +x "$HOME/bin/docker-machine.exe"
```
查看是否安装成功：
```
$ docker-machine version
```

- 启动

1. 启动对应虚拟机
1. 点击桌面上 Docker Quickstart Terminal 图标。


## docker-machine 常用命令

Docker Machine是一个简化Docker安装的命令行工具，通过一个简单的命令行即可在相应的平台上安装Docker

```
$ docker-machine ls #列出可用的机器
$ docker-machine create --driver virtualbox test #创建机器
$ docker-machine ip test #查看机器ip
$ docker-machine stop test #停止机器
$ docker-machine start test #启动机器
$ docker-machine ssh test #进入机器
$ docker-machine active #查看当前激活状态的docker主机
```

## docker-machine 挂载本地文件夹

Windows 和 MacOS 不能原生地支持 Docker ，所以需要先启动一个 docker machine ，然后在里面运行 Docker 。所以 docker machine 实际上就是一个虚拟机，通过 VirtualBox 可以进行配置。

那么问题来了， docker machine 是一个虚拟机，所以其内部的文件系统与外部是分离的，也就是说，从 Docker 里面是无法直接访问到物理机的文件系统的。正常情况下，通过 Volume 可以将文件系统挂载到 Docker 内部，但是在 docker machine 中运行的 Docker 则只能访问到 docker machine 里的文件系统。

1. 在本地创建共享目录
注意：不要在`C:\Users\...`目录下创建，否则会挂载失败
1. 点击`VirtualBox`中对应虚拟机，点击右侧设置，配置对应共享目录
2. 在 `docker machine` 里将共享的文件夹 `mount` 到 `虚拟机目录`
执行如下命令：

    ```
    sudo mount -t vboxsf <本地目录> <虚拟机目录>
    ```
注意：`<本地目录>`,` <虚拟机目录>`名称不要相同

## 利用Dockerfile部署SpringBoot项目
1. 创建`Dockerfile`
    
    ```
    FROM java8
    MAINTAINER scyking
    VOLUME /root/jar
    ADD  base-1.0-SNAPSHOT.jar base.jar
    ENTRYPOINT ["nohup","java","-jar","/base.jar","&"]
    EXPOSE 8761
    ```
1. 构建镜像
把jar包和Dockerfile放在同一个目录下(如：`/root/jar`)执行下面命令
    
    ```
    docker build -t <镜像名称> .
    ```
命令参数：
- `-t`指定镜像名
- `.`表示Dockfile在当前路径

1. 启动容器

    ```
    docker run -d --name <容器名称> -p 8080:8080 <镜像名称>
    ```
命令参数：
- `run`后台运行
- `-p`公开指定端口号映射
- `--name`指定容器名
查看正在运行的容器：

    ```
    docker ps
    ```
    
1. 查看运行日志

    ```
    docker logs --tail=100 <容器名称>
    ```

## 常用命令

- 容器操作常用命令
    
    ```
    docker rm -f contain_id #删除某个容器
    docker rm -f $(docker ps -a -q) #删除所有容器
    
    docker ps -a #查看所有容器
    docker start <容器 ID> #启动一个已停止的容器
    docker stop <容器 ID> #停止一个容器
    docker restart <容器 ID> #重启一个容器
    # 使用-d参数，后台启动容器，使用以下两个命令进入
    docker attach <容器 ID> #进入一个容器，退出会导致容器停止
    docker exec <容器 ID> #进入一个容器，退出不会导致容器停止
    
    docker port <容器名> <端口号> #查看容器端口绑定情况
    ```
    查看容器内日志：
    ```
    $ docker logs [OPTIONS] CONTAINER
    Options:
        --details        显示更多的信息
    -f, --follow         跟踪实时日志
        --since string   显示自某个timestamp之后的日志，或相对时间，如42m（即42分钟）
        --tail string    从日志末尾显示多少行日志， 默认是all
    -t, --timestamps     显示时间戳
        --until string   显示自某个timestamp之前的日志，或相对时间，如42m（即42分钟）
    ```
    
- 镜像操作常用命令

    ```
    docker rmi -f imags_id #删除某个镜像
    docker rmi -f $(docker images) #删除所有镜像
    
    docker images # 查看所有镜像
    docker search <镜像名称> #查找镜像
    docker pull <镜像名称> #拉取镜像
    
    ```

## 问题

### 启动虚拟机报错信息如下

```
NtCreateFile(\Device\VBoxDrvStub) failed: 0xc000000034
STATUS_OBJECT_NAME_NOT_FOUND (0 retries) (rc=-101)
Make sure the kernel module has been loaded successfully.
```
原因：vboxdrv服务没有正常运行
解决：
1. 命令行下输入`sc.exe query vboxdrv`检测vboxdrv的运行状态。如果`STATE`不是`RUNNING`，则需要启动该服务。
2. 启动命令`sc start vboxdrv`。启动失败则重装服务。
3. 右击`C:\Program Files\Oracle\VirtualBox\drivers\vboxdrv\VBoxDrv.inf`文件，安装。
4. 如果是`RUNNINF`状态，`sc delete newserv ` 删除服务重新安装

### 拉取镜像报`toomanyrequest...limit...`错误
解决：添加镜像提速

### 挂载本地文件夹错误
```
mounting failed with the error: Protocol error
```
解决：参考“docker-machine 挂载本地文件夹”中注意事项



     







