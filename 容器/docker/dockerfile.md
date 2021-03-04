# dockerfile

标签（空格分隔）： docker

---

[toc]

## 作用
用于构建镜像

## 指令

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






