# git 升级版本

标签（空格分隔）： git

---

[toc]

## 概述

> CentOS 系统自带git版本过低(1.18.3)，需要升级。

## 安装依赖

```bash
yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel asciidoc
yum install  gcc perl-ExtUtils-MakeMaker
```

## 卸载git

```bash
yum remove git
```

## 下载git

```bash
cd /usr/local/src/
wget https://www.kernel.org/pub/software/scm/git/git-2.31.1.tar.xz
```

## 编译安装

```bash
tar -vxf git-2.31.1.tar.xz
cd git-2.31.1
make prefix=/usr/local/git all 
make prefix=/usr/local/git install
## 配置环境变量
echo "export PATH=$PATH:/usr/local/git/bin" >> /etc/profile
source /etc/profile
```

## 查看版本

```bash
## 查看版本
[root@localhost git-2.31.1]# git --version
git version 2.31.1
## 查看安装路径
[root@localhost git-2.31.1]# which git
/usr/local/git/bin/git
```
