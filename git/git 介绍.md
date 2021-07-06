# git 介绍

标签（空格分隔）： git

---

[toc]

## 简介

> 是一个开源的分布式版本控制系统，用于敏捷高效地处理任何或小或大的项目。

1. Git 与 SVN 区别

- Git 是分布式的，SVN 不是。
- Git 将内容按元数据方式存储，而 SVN 是按文件。
- Git 分支和 SVN 的分支不同（SVN 的分支就是版本库中的另外一个目录）。
- Git 没有一个全局的版本号，而 SVN 有。
- Git 的内容完整性要优于SVN（Git的内容存储使用的是SHA-1哈希算法。这能确保代码内容的完整性，确保在遇到磁盘故障和网络问题时降低对版本库的破坏）。

1. Git 仓库

- 本地仓库
- 远程仓库

1. Git 常用操作

- git clone：将远程的Master分支代码克隆到本地仓库
- git checkout：切出分支出来开发
- git add：将文件加入库跟踪区
- git commit：将库跟踪区改变的代码提交到本地代码库中
- git push： 将本地仓库中的代码提交到远程仓库

## 使用

### 准备

- 安装 git 客户端 -- [download](https://git-scm.com/download/win)
- 已有相关Git账号 -- [github](https://github.com)、[码云](https://gitee.com/)

### git 配置

1. SSH 服务配置

    SSH（Secure Shell缩写）， 是专为远程登录会话和其他网络服务提供的安全性协议。

    - 生成SSH秘钥对

        打开 Git Bash 工具，输入如下命令：

        ```bash
        ssh-keygen -t rsa -C "邮箱地址"
        ```

        按提示完成SSH秘钥对生成。

    - 登录 github ， 添加 SSH keys
        - 依次点击 Settings --> SSH and GPG keys --> New SSH key
        - 添加上面生成的秘钥（.pub结尾文件）内容

1. 全局用户名及邮箱配置

    ```bash
    git config --global user.name "用户名"
    
    git config --global user.email "邮箱地址"
    ```

    PS:全局配置信息也可在 `.gitconfig` 文件中查看及修改。

1. 查看已有配置信息

    ```bash
    git config --list
    ```

## 解决github访问缓慢

> 配置系统host文件。

1. 配置 [github.com](https://github.com.ipaddress.com/)
1. 配置 [assets-cdn.github.com](https://github.com.ipaddress.com/assets-cdn.github.com)
1. 配置 [github.global.ssl.fastly.net](https://fastly.net.ipaddress.com/github.global.ssl.fastly.net#ipinfo)
1. cmd 下刷新系统DNS缓存，执行 `ipconfig /flushdns`（win10）

示例：

```hosts
# fix git clone github project failed 
140.82.113.3 github.com
140.82.112.3 github.com

185.199.108.153 assets-cdn.github.com
185.199.109.153 assets-cdn.github.com
185.199.110.153 assets-cdn.github.com
185.199.111.153 assets-cdn.github.com

199.232.69.194 github.global.ssl.fastly.net
```
