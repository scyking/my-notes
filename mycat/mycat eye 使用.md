# mycat eye 使用

标签（空格分隔）： mycat zookeeper

---

## zookeeper 安装

### 概述

一个分布式的，开放源码的分布式应用程序协调服务。提供的功能包括：配置维护、域名服务、分布式同步、组服务等。

### 安装步骤

- [下载地址](http://archive.apache.org/dist/zookeeper/)，下载后解压至相应目录
- 将 `..\conf` 目录下 `zoo_sample.cfg` 修改为 `zoo.cfg`
- 启动
    - windows： `..\bin\zkServer.cmd`
    - linux：`..\bin\zkServer.sh start`

## mycat eye 使用

### mycat eye 安装步骤

> mycat、mycat eye 依赖 jdk 1.7+；mycat eye  需要 Zookeeper 作为配置中心。

- [下载地址](https://github.com/MyCATApache/Mycat-download/tree/master/mycat-web-1.0)，下载指定版本
- 启动（需先启动 `zookeeper`）
    - windows：`..\start.bat`
    - linux：`..\start.sh`
- 访问地址：<http://localhost:8082/mycat/>

### 维护mycat节点

新增一个 mycat 服务，包括名称，IP，端口，数据库名称，用户名和密码。

### ~~维护mycat jmx信息~~