# docker apk 使用

---

[toc]

## 概述

> apk 是alpine包管理工具。
> Alpine中软件安装包的名字可能会与其他发行版有所不同，可以在<https://pkgs.alpinelinux.org/packages> 网站搜索并确定安装包的名称。

```bash
# 使用不在主索引内安装包方式
echo "http://dl-4.alpinelinux.org/alpine/edge/testing" >> /etc/apk/repositories
apk --update add --no-cache
```

## 安装

### 安装最新版本

```bash
apk add openssh openntp vim
apk add --no-cache mysql-client
# 指定第三方仓库
apk add docker --update-cache --repository http://mirrors.ustc.edu.cn/alpine/v3.4/main/ --allow-untrusted
```

### 安装指定版本

```bash
apk add asterisk=1.6.0.21-r0
apk add asterisk<1.6.1
apk add asterisk>1.6.1
```

## 其他命令

```bash
apk update                                      #更新最新本地镜像源
apk upgrade                                     #升级软件
apk add --upgrade busybox                       #指定升级部分软件包
apk search                                      #查找所以可用软件包
apk search -v                                   #查找所有可用软件包及其描述内容
apk search -v 'acf*'                            #通过软件包名称查找软件包
apk search -v -d 'docker'                       #通过描述文件查找特定的软件包
apk info                                        #列出所有已安装的软件包
apk info -a zlib                                #显示完整的软件包信息
apk info --who-owns /sbin/lbu                   #显示指定文件属于的包
apk add --allow-untrusted /path/to/file.apk     #本地安装
```

## apk 镜像站

- <http://dl-cdn.alpinelinux.org/alpine/>
- <http://nl.alpinelinux.org/alpine/>
- <http://uk.alpinelinux.org/alpine/>
- <http://dl-2.alpinelinux.org/alpine/>
- <http://dl-3.alpinelinux.org/alpine/>
- <http://dl-4.alpinelinux.org/alpine/>
- <http://dl-5.alpinelinux.org/alpine/>
- <http://dl-8.alpinelinux.org/alpine/>
- <http://mirror.yandex.ru/mirrors/alpine/>
- <http://mirrors.gigenet.com/alpinelinux/>
- <http://mirror1.hs-esslingen.de/pub/Mirrors/alpine/>
- <http://mirror.leaseweb.com/alpine/>
- <http://repository.fit.cvut.cz/mirrors/alpine/>
- <http://alpine.mirror.far.fi/>
- <http://alpine.mirror.wearetriple.com/>
- <http://mirror.clarkson.edu/alpine/>
- <http://linorg.usp.br/AlpineLinux/>
- <http://ftp.yzu.edu.tw/Linux/alpine/>
- <http://mirror.aarnet.edu.au/pub/alpine>
- <http://mirror.csclub.uwaterloo.ca/alpine>
- <http://ftp.acc.umu.se/mirror/alpinelinux.org>
- <http://ftp.halifax.rwth-aachen.de/alpine>
- <http://speglar.siminn.is/alpine>
- <http://mirrors.dotsrc.org/alpine>
- <http://ftp.tsukuba.wide.ad.jp/Linux/alpine>
- <http://mirror.rise.ph/alpine>
- <http://mirror.neostrada.nl/alpine/>

## 镜像源

- [清华TUNA镜像源](https://mirror.tuna.tsinghua.edu.cn/alpine/)
- [中科大镜像源](http://mirrors.ustc.edu.cn/alpine/)
- [阿里云镜像源](http://mirrors.aliyun.com/alpine/)

### 添加镜像地址

> 修改 `/etc/apk/repositories` 文件，直接添加源地址即可。

```bash
vi /etc/apk/repositories

# /media/cdrom/apks
http://mirrors.ustc.edu.cn/alpine/v3.5/main
http://mirrors.ustc.edu.cn/alpine/v3.5/community
```
