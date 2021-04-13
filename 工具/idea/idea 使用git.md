# idea 使用git

标签：idea git

---

一、Git 简介

Git是一个开源的分布式版本控制系统，用于敏捷高效地处理任何或小或大的项目。

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

二、准备

- 安装 git 客户端 -- [download](https://git-scm.com/download/win)
- 已有相关Git账号 -- [github](https://github.com)、[码云](https://gitee.com/)

三、git 配置

1. SSH 服务配置

    SSH（Secure Shell缩写）， 是专为远程登录会话和其他网络服务提供的安全性协议。

    - 生成SSH秘钥对

        打开 Git Bash 工具，输入如下命令：
    
        ```
        ssh-keygen -t rsa -C "邮箱地址"
        ```
        按提示完成SSH秘钥对生成。
    
    - 登录 github ， 添加 SSH keys
        - 依次点击 Settings --> SSH and GPG keys --> New SSH key 
        - 添加上面生成的秘钥（.pub结尾文件）内容

1. 全局用户名及邮箱配置
    
    ```
    $ git config --global user.name "用户名"
    
    $ git config --global user.email "邮箱地址"
    ```
    PS:全局配置信息也可在 `.gitconfig` 文件中查看及修改。

1. 查看已有配置信息
    
    ```
    git config --list
    ```

三、idea 配置

1. 配置 Git

    依次点击 File --> Setting --> Version Control --> Git，进行配置。
    
    ![Git 配置页面](https://images2018.cnblogs.com/blog/793754/201809/793754-20180913181112848-1206585689.png)
    
1. 配置 GitHub

    依次点击 File --> Setting --> Version Control --> GibHub，进行配置。
    
    ![Github 配置页面](https://img2018.cnblogs.com/blog/793754/201809/793754-20180913204843803-394725031.png)

1. 添加 Git Repository

    - 本地仓库创建
        - 创建项目
        - 依次点击 VCS --> Import into Version Control --> Create Git Repository
        - 选择本地仓库位置
    
    - 文件颜色含义
        - 褐色：未add到git管理
        - 绿色：已add到git管理
        - 蓝色：文件有修改
        - 白色（普通色）：已提交文件

    - 应用远程仓库
        - 项目第一次 `push`,配置已有远程仓库地址
        ![配置页面](https://img2018.cnblogs.com/blog/793754/201809/793754-20180913221420622-1615921079.png)
        - 将项目上传至远程仓库
        依次点击 VCS --> Import into Version Control --> Share Project on GitHub
        - 从远程仓库克隆项目
        依次点击 File --> New --> Project from Version Control --> Git
        
        
    


    
