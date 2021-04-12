# docker 常用命令

标签（空格分隔）： docker

---
[toc]

## docker-machine

```
    docker-machine ls #列出可用的机器
    docker-machine create --driver virtualbox test #创建机器
    docker-machine  rm test # 删除机器
    docker-machine ip test #查看机器ip
    docker-machine stop test #停止机器
    docker-machine start test #启动机器
    docker-machine ssh test #进入机器
    docker-machine active #查看当前激活状态的docker主机
```

## container-容器相关
    
```
    docker run -d --name <容器名称> -p 8080:8080 <镜像名称>
    
    docker rm -f <container_id> #删除某个容器
    docker rm -f $(docker ps -a -q) #删除所有容器
    
    docker ps -a #查看所有容器
    docker start <container_id> #启动一个已停止的容器
    docker stop <container_id> #停止一个容器
    docker restart <container_id> #重启一个容器
    # 使用-d参数，后台启动容器，使用以下两个命令进入
    docker attach <container_id> #进入一个容器，退出会导致容器停止
    docker exec <container_id> #进入一个容器，退出不会导致容器停止
    
    docker port <容器名> <端口号> #查看容器端口绑定情况
    docker inspect <CONTAINER> | grep IPAddress #查看容器ip地址
```
    
## image-镜像相关

```
    docker build -t <镜像名称> . # 构建镜像 .表示与dockerfile在同一目录

    docker rmi -f <image_id> #删除某个镜像
    docker rmi -f $(docker images) #删除所有镜像
    
    docker images # 查看所有镜像
    docker search <镜像名称> #查找镜像
    docker pull <镜像名称> #拉取镜像
    
    $ docker rmi $(docker images | grep "none" | awk '{print $3}') # 删除标签是none的镜像
```

## docker-compose

```
    docker-compose up   #启动
    -d                  后台启动
    -f                  指定yml文件，默认是当前目录下docker-compose.yml
    --build             重新构建镜像
    docker-compose down #停止所有容器
    --rmi all           删除所有镜像
```

## 其他
### 查看容器内日志
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
### 镜像导入导出
1. 命令
- save
    ```
    docker save [options] images [images...]
    Options:
    -o, --output [目标文件]     输出到文件
    >  [目标文件]               输出到文件
    ##示例
    docker save -o nginx.tar nginx:latest
    docker save > nginx.tar nginx:latest
    ```
    
- load
    ```
    docker load [options]
    Options:
    -i, --input [目标文件]
    -q, --quit  
    ##示例
    docker load -i nginx.tar
    docker load < nginx.tar
    ```
- export
    ```
    docker export [options] container
    Options:
    -o, --output [目标文件]
    ##示例
    docker export -o nginx-test.tar nginx-test
    ```
- import
    ```
    docker import [options] file|URL|- [REPOSITORY[:TAG]]
    Options:
    -c, --change list
    -m, --message string 
    ##示例
    docker import nginx-test.tar nginx:imp
    cat nginx-test.tar | docker import - nginx:imp
    ```
- 为名称、标签为`<none>`的镜像添加名称
    ```
    docker tag [image_id] [name:tag]
    ## 示例
    docker tag eb40dcf64078 django:latest
    ```

1. 区别
- export命令导出的tar小于save命令导出的
- export命令是从容器中导出tar文件，而save命令是从镜像中导出
- export导出的文件再import时，无法保留镜像所有历史
1. 建议
- 备份images，使用save、load
- 启动容器后，容器内容有变化，需要备份，使用export、import







