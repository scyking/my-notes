# docker 使用

标签：docker

---

[toc]

## 基本概念

- `image`：镜像，相当于是一个root文件系统。
- `container`：容器，就像是面向对象程序设计中的实例和类一样。镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。
- `repository`：仓库，可看成一个代码控制中心，用来保存镜像。

## windows 上使用docker

### 安装

- [下载Docker Toolbox](http://mirrors.aliyun.com/docker-toolbox/windows/docker-toolbox/)
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

### docker-machine 挂载本地文件夹

1. 在本地创建共享目录
注意：不要在`C:\Users\...`目录下创建，否则会挂载失败
1. 点击`VirtualBox`中对应虚拟机，点击右侧设置，配置对应共享目录
1. 在 `docker machine` 里将共享的文件夹 `mount` 到 `虚拟机目录`
    执行如下命令：

    ```
    sudo mount -t vboxsf <本地目录> <虚拟机目录>
    ```
    
    PS：`<本地目录>`,` <虚拟机目录>`名称不要相同

### 问题

#### 启动虚拟机报错信息如下

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

#### 拉取镜像报`toomanyrequest...limit...`错误
解决：添加镜像提速

#### 挂载本地文件夹错误
```
mounting failed with the error: Protocol error
```
解决：`<本地目录>`,` <虚拟机目录>`名称不要相同

## dockerfile 指令

|指令|说明|
|:---:|:---:|
|FROM|设置镜像使用的基础镜像
|MAINTAINER|设置镜像作者
|RUN|编译镜像时运行的脚本
|CMD|设置容器的启动命令
|LABEL|设置镜像的标签
|EXPOSE|设置镜像暴露的端口
|ENV|设置容器的环境变量
|ADD|编译镜像时复制文件到镜像中
|COPY|编译镜像时复制文件到镜像中
|ENTRYPOINT|设置容器的入口程序
|VOLUME|设置容器的挂载卷
|USER|设置运行RUN CMD ENTRYPOINT的用户名
|WORKDIR|设置RUN CMD ENTRYPOINT COPY ADD指令的工作目录
|ARG|设置编译镜像时加入的参数
|ONBUILD|设置镜像的ONBUILD指令
|STOPSIGNAL|设置容器的退出信号量